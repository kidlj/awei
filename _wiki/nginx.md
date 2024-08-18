---
title: Nginx
---

### 安装

安装 Nginx 一般采用源码编译的方式。Nginx 采用模块化设计，一些默认自动编译的模块可以通过 `--without-` 参数禁用，一些内置模块中的附加模块可以通过 `--with-` 参数手动开启。具体有哪些参数可以通过 `./configure --help` 命令列出。另外，编译 Nginx 是还可以通过 `--add-module=/path/module1` 的方式编译第三方模块。

一次示例安装：

	# yum install pcre pecre-devel openssl openssl-devel zlib zlib-devel 
	
	# ./configure \
		--prefix=/usr/local/nginx \
		--with-http_ssl_module \
		--add-module=../ngx_http_google_filter_module/

	# make && make install

### 启动和控制 Nginx

Nginx 有一个主进程和多个 worker 进程。主进程的作用是读取配置文件并且维护 worker 进程，worker 进程用来实际地处理请求。

要启动 Nginx，简单地执行二进制文件 `./nginx` 就好。Nginx 启动以后，可以用向主进程发送信号：

	# nginx -s signal

信号可以是：

* `stop` -- fast shutdown 
* `quit` -- graceful shutdown
* `reload` -- 重载配置文件
* `reopen` -- 重开日志文件

配置文件发生改变以后必须用 `-s reload` 来重新加载或者重启 Nginx 才会生效。当主进程收到 reload 信号以后，它会检查配置文件的有效性，如果有效，那么主进程启动新的 worker 进程来处理请求，同时向旧的 worker 进程发送信息要求它们关闭。否则仍然使用旧的进程和旧的配置。旧的 worker 进程收到主进程发送的关闭信号以后，立即停止接受新的请求，直到把现有请求处理完，它们就会退出。

另外还可以使用 Unix 工具比如 kill 向 Nginx 发送信号：

	# kill -s QUIT `cat /usr/local/nginx/logs/nginx.pid`


### 配置文件的结构

Nginx 的主配置文件在 `/usr/local/nginx/conf/nginx.conf`。在配置文件里，Nginx 用“指令”来控制各个可用的模块。

指令包括简单指令和块指令两种。简单指令就是指令名字，后面加上用空格分隔的指令参数，最后以分号结尾。块指令跟简单指令类似，不过后面不是以分号而是以一对大括号 `{ }` 结尾。如果一个块指令的大括号里面可以包含有其它指令，就可以称它为 context。

不在某个 context 里的指令一定就在 main context 里，比如 `event` 和 `http` 指令。`server` 指令在 `http` context 里，`location` 指令在 `server` context 里。

注释以 `#` 开头。

### 伺服静态文件

通常，配置文件里包含多个 `server` 指令，它们定义了侦听不同端口和匹配不同主机名的虚拟主机。当 Nginx 决定了用哪一个虚拟主机来处理一个请求以后，它会将从该请求里头部获取到的 URI 与 该 server 中定义的 `location` 后面的参数做一个对比。

现在添加如下的配置：

	server {
		location / {
			root /data/www
		}

		location /images/ {
			root /data
		}
	}

`location` 指令定义了要和发过来的请求 URI 做对比的 prefix。当一个请求的 URI 的开始部分和一个 prefix 匹配，那么就将该 URI 添加到 `root` 指令的参数定义的路径后面，组成一个新的路径，也就从主机文件系统上的这个路径来获取请求的静态文件。

举例来说，当一个 `http://localhost/index.html` 的请求发过来时，获取到的 URI 是 `/index.html`，它与 `location /` 这个 prefix 相匹配，那么就组成一个新的路径 `/data/www/index.html`，以系统上的这个文件来响应这个请求。 

当有多个 prefix 与 请求 URI 相匹配时，那么使用最长匹配的那一个。上面配置中给出的 prefix `/` 其匹配长度为 1 且匹配任意一个请求，因此当其他所有 prefix 都不匹配时就会用到这一个匹配。

再比如对于 `http://localhost/images/nginx_logo.png` 这样的一个请求，`location /images/` 是最长匹配，Nginx 会提供文件系统上的 `/data/images/nginx_logo.png` 文件作为响应。如果该文件不存在，就会返回标志 404 错误的页面。

### Location 

Location 可以被定义为一个 prefix 字符串或者一个正则表达式。

匹配的顺序如下：

1. 按顺序检查 prefix；
2. 将最长的 prefix 存储起来；
3. 按顺序检查正则表达式，遇到第一个匹配的则停止检查，并使用相应的配置；
4. 如果所有的正则表达式都不匹配，那么使用之前存储的那个最长 prefix 对应的配置。

这里有两个例外：

1. 如果上面第 2 步存储的最长 prefix 之前有 `^~` 符号，则不会检查正则表达式；
2. 可以用 `=` 定义完全匹配，如果发现一个完全匹配，则检查终止。