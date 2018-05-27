---
title: net
---

net/http
========

### type Handler

    type Handler interface {
        ServeHTTP(ResponseWriter, *Request)
    }

### type HandlerFunc

    // The HandlerFunc type is an adapter to allow the use of
    // ordinary functions as HTTP handlers. If f is a function
    // with the appropriate signature, HandlerFunc(f) is a
    // Handler that calls f.
    type HandlerFunc func(ResponseWriter, *Request)

    // ServeHTTP calls the underlyinig function
    func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
        f(w, r)
    }

`HandlerFunc` 不是一个函数，而是一个 named type。它可以将一个符合签名的函数或 method value 转型为满足 `Handler` 接口的对象（这里是函数）。

Example:

    mux := http.NewServeMux()
    mux.Handle("/list", http.HandlerFunc(db.list)) // db.list 是一个 method value
