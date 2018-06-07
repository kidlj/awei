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

Example:

    mux := http.NewServeMux()
    mux.Handle("/list", http.HandlerFunc(db.list)) // db.list 是一个 method value

### func FileServer

    // wrapper
    func FileServer(root FileSystem) Handler {
        return &fileHandler{root}
    }

Example:

    files := http.FileServer(http.Dir("static"))
    mux.Handle("/static", http.StripPrefix("/static", files))

### type fileHandler


    type fileHandler struct{
        root FileSystem
    }

    func (f *fileHandler) ServeHTTP(w ResponseWriter, r *Request) {
        upath := r.URL.Path
        if !strings.HasPrefix(upath, "/") {
            upath = "/" + upath
            r.URL.Path = upath
        }
        serveFile(w, r, f.root, path.Clean(upath), true)
    }

### func serveFile

    func serveFile(w ResponseWriter, r *Request, fs FileSystem, name string, redirect bool) {
        ...
        f, err := fs.Open(name)
        ...
    }

### type Dir

    type Dir string

    func (d Dir) Open(name string) (File, error) {...}

### type FileSystem

    type FileSystem interface{
        Open(name string) (File, error)
    }

### func StripPrefix

    func StripPrefix(prefix string, h Handler) Handler {
        if prefix == "" {
            return h
        }

        return HandlerFunc(func (w ResponseWriter, r *Request) {
            if p := strings.TrimPrefix(r.URL.Path, prefix); len(p) < len(r.URL.Path) {
                r.URL.Path = p
                h.ServeHTTP(w, r)
            } else {
                NotFound(w, r)
            }
        })
    }
