---
title: Next.js 服务端渲染请求透传
---

Next.js 中一个页面就是一个 React Component，而服务端渲染（SSR）则被简化为在页面被请求时提供给组件一个 Props 对象。页面的 Props 对象由 Next.js server 根据请求数据动态提供，这个过程中可以与后端 API server 交互以及获取数据。很多场景下，一个 Next.js 站点的页面由客户端渲染和服务端渲染混合组成，客户端代码通过 cookie 认证用户请求，而我们希望服务端渲染通过 `getServerSideProps` 机制获取 Props 数据的过程中，能够透传客户端的请求 cookie 到后端 API server，这样就不用专门为服务端渲染维护一套认证逻辑。

客户端请求与服务端渲染请求的认证逻辑如下：

Fig 1. 客户端渲染

```
		   cookie
[client]	--> 	[API server]
```


Fig 2. 默认服务端渲染

```
		   cookie					 no cookie
[client] 	--> 	[Next.js server] 	--> 	[API server]
```

Fig 3. 我们希望达成的服务端渲染过程

```
		   cookie					 same cookie
[client] 	--> 	[Next.js server] 	--> 	[API server]
```

每个 Next.js 服务端渲染页面都需要定义 `getServerSideProps` 函数，在这里从后端 API 获取数据并返回给组件作为渲染 Props。Next.js server 向后端 API 获取数据时可以使用和客户端一样的请求函数库，比如原生的 fetch 或者 axios 等。区别是默认情景下，客户端 fetch 会传递 cookie，而服务端的 fetch 是没有 cookie 的。不过在 `getServerSideProps` 函数中，我们可以从 `context` 参数中获取到一次页面请求的所有数据，也包括 cookie。所以要透传 cookie 只需要将客户端的 cookie 传递给服务端请求即可。一个获取用户数据的典型示例如下（`fetchData` 是对原生 `fetch` 工具的范型封装，函数签名是一样的）：

```typescript
export const getServerSideProps: GetServerSideProps<FetchResult<IUser>> = async context => {
	const cookie = context.req.headers.cookie
	const { data, error } = await fetchData<IUser>(sessionEndpoint, {
		headers: {
			cookie: cookie!
		}
	})
	if (error && error.errorCode == 401) {
		return {
			redirect: {
				destination: '/account/login?next=/',
				permanent: false,
			}
		}
	}
	return {
		props: {
			data: data || null,
			error: error
		}
	}
}
```

更进一步地，如果我们想透传一个页面请求的所有 headers，只需对上述函数做一些修改即可：

```typescript
export const getServerSideProps: GetServerSideProps<FetchResult<IUser>> = async context => {
	const { data, error } = await fetchData<IUser>(sessionEndpoint, {
		headers: context.req.headers as Record<string, string>
	})
	if (error && error.errorCode == 401) {
		return {
			redirect: {
				destination: '/account/login?next=/',
				permanent: false,
			}
		}
	}
	return {
		props: {
			data: data || null,
			error: error
		}
	}
}
```

透传所有 headers 的一个应用场景是，我们想给 API 设置 rate limiting，不论这个请求是客户端发起的还是 Next.js server 发起的。而 rate limiting 通常是根据请求 IP 来限定的，比如限定每个 IP 每秒请求次数不超过 10 个；客户端请求自然有不同的 IP，而 Next.js server 发起的请求总是其所在的机器 IP。我们希望服务端渲染也能透传客户端的真实 IP，否则这样的 rate limiting 就是错误的。

假设我们用 Nginx 来负载均衡 Next.js server 和 API server。不论是客户端直接请求 API server 还是服务端渲染时 Next.js server 请求 API server，地址都指向 Nginx，供 Next.js server 请求的 API 专门设置一个 Nginx server，在 Next.js 项目里设置服务端渲染的 API 地址时指向这个内网地址和端口。一个典型的配置如下：

```nginx.conf
http {
	# 客户端 API 请求的 rate limiting zone
	limit_req_zone $binary_remote_addr zone=backend:10m rate=10r/s;
	# SSR API 请求的 rate limiting zone; 只使用一个 zone 会导致 100% 503，原因待调查
	limit_req_zone $binary_remote_addr zone=ssr-backend:10m rate=10r/s;

    upstream metwords-frontend {
        ip_hash;
        keepalive 16;

        server code:3000;
        server cache:3000;
    }

    upstream metwords-backend {
        ip_hash;
        keepalive 16;

        server code:8080;
        server cache:8080;
    }

    server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  www.metwords.com;
        server_name  metwords.com;
        root         /usr/share/nginx/html;

		location / { 
            root   html;
            index  index.html index.htm;
			# 透传客户端真实 IP 给 Next.js server
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_http_version 1.1;
            proxy_set_header Connection "";

            proxy_pass http://metwords-frontend;
        }   


		# 供客户端请求的 API 地址
        location /api/ {
            real_ip_header X-Forwarded-For;
            set_real_ip_from  172.21.0.0/20;
            set_real_ip_from  127.0.0.1;
            proxy_http_version 1.1;
            proxy_set_header Connection "";

			# 设置 rate limiting
            limit_req zone=backend nodelay;

            proxy_pass http://metwords-backend/;
        }
	}

	server {
        listen 8081;

		# 供 Next.js server 请求的 API 地址
        location /api/ {
            real_ip_header X-Forwarded-For;
			# 从 Next.js server 取出客户端真实 IP
            set_real_ip_from  172.21.0.0/20;
            set_real_ip_from  127.0.0.1;
            proxy_http_version 1.1;
            proxy_set_header Connection "";

			# 设置 rate limiting
            limit_req zone=ssr-backend nodelay;

            proxy_pass http://metwords-backend/;
        }
    }
}

```

上述配置中，服务端渲染页面的请求链路是这样子的：

Fig 4. 使用 Nginx 负载均衡 Next.js server 和 API server 后的服务端渲染请求链路

```
real IP		set X-Forwarded-For		 with headers	   get real IP
[client] -->	 [Nginx] 		--> [Next.js server] --> [Nginx] --> [API server]
```

对比文章开头 Fig 3 列出的服务端渲染链路，这时链路中间增加了两次 Nginx 转发。第一次 Nginx 转发中，我们将客户端 IP 设置成 X-Forwarded-For header 传递给 Next.js server，Next.js server 请求 Nginx 时会带上这个 header（代码给 fetchData 设置了 with headers）。第二次 Nginx 转发使用 read_ip_module 的 `set_real_ip_from` 指令从 Next.js server 的 X-Forwarded-For header 中取出客户端真实 IP，Nginx 接着会将 $binary_remote_addr 变量设置成这个真实 IP，然后使用这个变量值作为 key 进行 rate limiting。

这里还需要记录一个问题，实验中发现，如果只设置一个 rate limit zone 供客户端请求的 /api 和供 Next.js server 请求的 /api 一起使用，Next.js 请求 /api 会 100% 出现 503（服务受限）的情况，如示例中一样使用两个分开的 rate limit zone 工作正常，原因未知。先记录下待以后阅读了 Nginx rate limiting 部分的源码再来了解。