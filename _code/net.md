---
title: net
---

net/http
========

### HandlerFunc

    // The HandlerFunc type is an adapter to allow the use of
    // ordinary functions as HTTP handlers. If f is a function
    // with the appropriate signature, HandlerFunc(f) is a
    // Handler that calls f.
    type HandlerFunc func(ResponseWriter, *Request)

    // ServeHTTP calls the underlyinig function
    func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
        f(w, r)
    }

`HandlerFunc` 不是一个函数，而是一个 named type。它可以将一个符合签名的函数转型为满足 `Handler` 接口的对象（这里是函数）。

    type Handler interface {
        ServeHTTP(ResponseWriter, *Request)
    }
