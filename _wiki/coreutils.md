---
title: CoreUtils
---

### tar

列出查看

    $ tar -tf latest.tar.gz

打包时不要前导路径，比如将`~/GitHub/kidlj.github.io/_site/`目录下的文件打包：

	$ cd ~/GitHub/kidlj.github.io/_site
	$ tar -cf site.tar .

或者，

	$ tar -cf site.tar -C ~/GitHub/kidlj.github.io/_site/ .

如何在解压时剥落压缩包内前导路径？

    $ pass

分卷压缩

    $ tar zcf - 2017.log |split -d -b 1000m - logs.tar.gz.
    $ cat logs.tar.gz* | tar zx


### lsof

查看某个进程打开了哪些文件：

	$ lsof -p 1489

列出哪个程序在占用某个端口：

	$ lsof -i :22


### find

#### 通配符扩展

`-name` 可以使用 shell 通配符扩展，不过要用括号括起来，以免被 shell 拦截解释。

	$  find . -name '*.txt'

#### 正则表达式

Find 支持多种正则表达式实现，

	$ find . -regextype posix-extended -regex ".*[0-9]*.png"

但是需要注意的，`find`里的正则表达式搜索的是路径名，而不是文件裸名。

#### Or 条件操作

打印出 shell 文件和 python 文件：

	$ find . \( -name "*.sh" -o -name "*.py" \) -print

#### 否定参数

匹配不以 `.txt` 结尾的文件：

	$ find . ! -name "*.txt" -print


#### 基于目录深度的搜索

仅仅搜索一级目录：

	$ find ~/Downloads -maxdepth 1 -name "*.zip"

打印出深度距离起始目录至少两个子目录的文件：

	$ find ~/Downloads -mindepth 2 -name "*.pdf"

#### 基于文件类型搜索

	$ find . -type f -name '*.py' -print

#### 基于文件时间搜索

- 访问时间(-atime): 最近一次访问文件的时间
- 修改时间(-mtime): 内容最后一次被修改的时间
- 变化时间(-ctime): 文件元数据（如权限或所有权）最后一次改变的时间

找出访问时间在 1 天前的文件：

	$ find . -atime 1 -print

找出修改时间距现在 6 天内的可执行文件：

	$ find . -mtime -6 -type f -executable

找出比 `file.txt` 修改时间更近的文件：

	$ find . -newer file.txt -print

#### 基于文件大小的搜索

	$ find . -type f -size +2k

#### 基于文件权限和所有权的搜索

	$ find . -type f -name "*.py" ! -perm 644 -print
	$ find . -type f -user root -print


#### 删除匹配的文件

	$ find . -type f -name "*.swp"

####  执行命令或操作

每个 `-exec` 只能跟一个命令（或脚本），必须以 `;` 作为结束，而且尽管可能会执行后面的命令多次，但整个 `find` 命令最终只输出一个数据流。

	$ find . -type f -user root -exec chown mellon {} \;

如果把 `;` 换成 `+`，那么后面的命令只会执行一次，把所有匹配的文件追加作为它的运行参数。

	$ find . -type f -exec echo {} +

####  跳过特定的目录

	$ find . \( -name ".git" -prune \) -o \( -type f -print \)

### xargs

#### 指定输出列数：

	$ cat data.txt | xargs -n 

#### 指定界定符：

	$ echo "splitXsplitXsplit" | xargs -d X

#### 执行命令并传递参数

xargs 能把从 stdin 接收到的数据重新格式化，再将其作为参数提供给其它命令（附在命令之后）：

	$ cat args.txt | xargs -n 1 echo

#### 传递特定位置的参数

	$ cat args.txt | xargs -I {} ./script.sh -p {} -1

对于获取到的每一个参数，`xargs` 后的命令对应地执行一次：

	$ cat args.txt | xargs 
	l lh li
	$ cat args.txt | xargs -I {} ls -{} .

此处`ls` 命令会执行 3 次。

#### 结合 find 使用 xargs

有一种常见的错误的组合方式来使用它们应该极力避免：

	$ find . -type f -name "*.txt" -print | xargs rm -f

这是做很危险，有时会删除不必要删除的文件。如果输出的某个文件名包含空格，比如 `filename with whitespace.txt`，因为 `xargs` 采用的默认界定符包含空格， 这时 `rm` 将会尝试删除`filename`, `with` 和 `whitespace.txt`。

所以只要把 `find` 的输出结果作为 `xargs` 的输入，就必须将 `-print0` 与 `find` 结合使用，以字符 null(`'\0'`) 来分隔输出：

	$ find . -type f -name "*.txt" -print0 | xargs -0 rm -f

#### 另一种获取参数并执行命令的方式

包含 `while` 循环的子 shell 也可以用来由 `stdin` 读取参数并执行命令：

	$ cat files.txt | ( while read arg; do cat $arg; done )

这等同于：

	$ cat file.txt | xargs -I {} cat {}

不过子 shell 的方式更加灵活，比如可以对同一个参数执行多条命令。


### Tr

`Tr` 只能通过 `stdin` 获取输入，它将输入的字符从 `set1` 映射到 `set2`，然后输出结果：

	$ tr [options] set1 set2

如果 `set2` 的长度不及 `set1`, 则用它的最后一个字符补齐；如果 `set2` 大于 `set1`，则多余的字符被忽略。

#### 转换字符

	$ echo 'This is a test' | tr 'a-zA-Z' 'n-za-mN-ZA-M'

#### 删除字符

删除在集合 `set1` 里的字符：

	$ echo 'This is a test' | tr -d '[set1]'

删除不在集合 `set1` 里的字符(删除它们的补集里的字符)：

	$ echo 'This is a test' | tr -d -c '[set1]'

#### 压缩字符

删除重复的空格：

	$ echo "GNU is not     UNIX.   Right? " | tr -s " "

删除多余的空行：

	$ cat file.txt | tr -s '\n'

#### 字符类

tr 支持标准字符类，比如：

	$ echo 'hello world' | tr '[:lower:]' '[:upper:]'


### Cut

字段默认界定符是 TAB。

#### 按字符切割

	$ echo "hello" | cut -c1
	h
	$ echo "hello | cut -c2-3
	el

#### 指定界定符和字段

	$ echo 12345.678 | cut -d. -f2
	678

	# 如果某行不含界定符，则打印该行，除非同时指定 `-s` 选项
	$ echo 12345 | cut -d. -f2
	12345

	$ echo 12345 | cut -s -d. -f2
	(empty output)

### echo

echo 一个变量的时候，变量一定要加上双引号。比如以下两个结果是不同的：

	$ var="  3"
	$ echo $var
	3
	$ echo "$var"
	3

### tee

自己编译安装软件时，记录安装到文件系统的文件以便以后卸载删除：

	$ sudo make install | tee make_install_manifest.txt

### diff

两个文件相同时，diff 的返回值是 0；不相同时，返回值是 1。

### Curl

#### Header

    $ curl -s -H "canary: true" http://test.domain.com/header

#### Cookie

    $ curl -s --cookie "canary=true" http://test.domain.com/cookie