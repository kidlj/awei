---
title: Node.js
---

### CWD

程序文件中用到的 `./` 和 `process.cwd()` 将总是返回运行 `node` 命令的那个目录，也是就文档所说的 *the working directory of the process*，而不是指该程序文件所在的目录[1]。

而`__dirname` 将总是指该程序文件所在的目录。

特别的是，用在 `require(./lib/user)` 中的 `./` 总是指该程序文件所在的目录。

因此在 Node 程序中，用到文件路径时一般（除了 `require`）不要用`./`，而应该用`__dirname` 来拼接。这样就会避免你在不同的目录执行程序入口文件可能会发生的文件路径错误。

### pm2

    # npm install -g pm2

    # cd IVNews
    # export ENV='production'
    # export PORT=80
    # pm2 start npm -- start // `npm start'
    # pm2 list

### npm

proxy:

    $ export npm_config_proxy=http://127.0.0.1:1080
    $ npm install

### EventEmitter
    
> The `eventEmitter.emit()` method allows an arbitrary set of arguments to be passed to the listener functions. It is important to keep in mind that when an ordinary listener function is called by the EventEmitter, the standard this keyword is intentionally set to reference the EventEmitter to which the listener is attached.[2]

> It is possible to use ES6 Arrow Functions as listeners, however, when doing so, the this keyword will no longer reference the EventEmitter instance.


### 中间件的 next()

中间件函数执行 `next()` 以后，该中间件函数并不会终止而是会继续执行完毕。

比如：

    function hello(req, res, next) {
        next();
        console.log('I was executed!');
    }

使用该中间件时，console 中将会输出 `I was executed!`。

同样地，即使是 `next(err)` 的情况该中间件也会执行完毕。所以一般调用到 `next(err)` 时，应该在其前面加上 `return` 来立即终止该中间件，否则可能会造成意外的错误，比如 *Error: Can't set headers after they are sent*。

### app.use() 和 app.get() 对路径处理的区别

简单来说，

- app.use('/path', someMiddleware) 匹配的是*所有以 '/path' 开头*的请求，像 '/path/more' 也会匹配。而且在这里会从 req.url 中去掉匹配前缀，再交给后续中间件处理。
- app.get('/path', someMiddleware) 匹配的*只有* '/path' 和 '/path/' 这两个，像 '/path/more' 都不会匹配。

虽然两者语法相似，但用处完全不同。app.use() 来源于 Connect[3] 的中间件挂载函数，而 app.get() 则是 Express 单独开发的路由匹配函数[4]。

[1]: http://stackoverflow.com/questions/8131344/what-is-the-difference-between-dirname-and-in-node-js
[2]: https://nodejs.org/docs/latest-v4.x/api/events.html#events_passing_arguments_and_this_to_listeners 
[3]: https://github.com/senchalabs/connect#mount-middleware
[4]: http://expressjs.com/en/guide/routing.html