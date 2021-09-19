---
title: Next.js SSR Request Passthrough - EN
---

A page in Next.js is a React Component, and server-side rendering (SSR) is reduced to providing the component with a Props object when the page is requested. The page's Props object is dynamically provided by the Next.js server based on the requested data, and this process allows interaction with the back-end API server and access to the data. In many cases, a Next.js site consists of a mix of client-side rendering and server-side rendering pages, where the client-side code authenticates the user request through a cookie, and we want the server-side rendering to pass the client-side request cookie to the back-end API server when it gets the Props data through the `getServerSideProps` mechanism. So that we don't have to maintain a set of authentication logic specifically for server-side rendering.

The authentication logic for client-side requests and server-side rendering requests is as follows.

Fig 1. Client side rendering

```
		   cookie
[client]	--> 	[API server]
```


Fig 2. Server side rendering by default

```
		   cookie					 no cookie
[client] 	--> 	[Next.js server] 	--> 	[API server]
```

Fig 3. The server-side rendering process we want to achieve

```
		   cookie					 same cookie
[client] 	--> 	[Next.js server] 	--> 	[API server]
```

Every Next.js server side rendered page needs to define the `getServerSideProps` function, where it fetches data from the backend API and returns it to the component as Props. next.js server can use the same request libraries as the client when fetching data from the backend API, such as the native fetch or axios, etc. The difference is that by default, client-side fetch passes cookies, while server-side fetch does not have a cookie. However, in the `getServerSideProps` function, we can get all the data of a page request from the `context` parameter, including the cookie, so to pass a cookie, we just need to pass the client-side cookie to the server-side request. A typical example of fetching user data is as follows (`fetchData` is a generic wrapper around the native `fetch` tool, the function signature is the same).

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

Further, if we want to pass through all the headers of a page request, we can simply make a few changes to the above function.

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

One application scenario where we want to pass all headers through is when we want to set rate limiting for the API, regardless of whether the request is initiated by the client or by the Next.js server. The rate limiting is usually based on the request IP, e.g., no more than 10 requests per second per IP; client requests naturally have different IPs, while requests from the Next.js server are always having the IP of the machine they are on. Such rate limiting would be wrong.

Suppose we use Nginx to load-balance the Next.js servers and the API servers. Whether the client requests the API server directly or the Next.js server requests the API server wtih server side renderings, we point the address to Nginx. The API requested by the Next.js server has a dedicated Nginx server, and the intranet address is set in the Next.js project for server side rendering pages. A typical configuration is as follows.

```nginx.conf
http {
	# rate limiting zone for client side api requests
	limit_req_zone $binary_remote_addr zone=backend:10m rate=10r/s;
	# rate limiting zone for server side rendering requests
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
			# Pass the real IP of the client to the Next.js server
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_http_version 1.1;
            proxy_set_header Connection "";

            proxy_pass http://metwords-frontend;
        }   


		# API address for client requests
        location /api/ {
            real_ip_header X-Forwarded-For;
            set_real_ip_from  172.21.0.0/20;
            set_real_ip_from  127.0.0.1;
            proxy_http_version 1.1;
            proxy_set_header Connection "";

			# set rate limiting
            limit_req zone=backend nodelay;

            proxy_pass http://metwords-backend/;
        }
	}

	server {
        listen 8081;

		# API address for Next.js server requests
        location /api/ {
            real_ip_header X-Forwarded-For;
			# Get the real IP of clients from Next.js server
            set_real_ip_from  172.21.0.0/20;
            set_real_ip_from  127.0.0.1;
            proxy_http_version 1.1;
            proxy_set_header Connection "";

			# set rate limiting
            limit_req zone=ssr-backend nodelay;

            proxy_pass http://metwords-backend/;
        }
    }
}

```

In the above configuration, the request chain for server side rendering pages looks like this.

Fig 4. Server-side rendering request chain after using Nginx to load balance Next.js servers and API servers

```
real IP		set X-Forwarded-For		 with headers	   get real IP
[client] -->	 [Nginx] 		--> [Next.js server] --> [Nginx] --> [API server]
```

Comparing to the server-side rendering chain listed in Fig 3 at the beginning of the article, two Nginx forwards are added to the chain. In the first Nginx forward, we set the client IP to an X-Forwarded-For header and pass it to the Next.js server, which takes this header with it when it requests Nginx (the code has set `fetchData` to request with headers). The second Nginx forward uses the read_ip_module's `set_real_ip_from` directive to retrieve the client's real IP from the Next.js server's X-Forwarded-For header, Nginx then sets the $binary_remote_addr variable to this real IP, and uses this variable as the key for rate limiting.

There is one issue to document here. We found in experiments that if we set only one rate limit zone for the client /api request and for the Next.js server /api request usage together, the Next.js server /api requests will be 100% 503 (service limited), but as in the example, using two separate rate limiting zones works fine, for reasons unknown. I'm logging this and will read the source code of the Nginx rate limiting part later to learn why.