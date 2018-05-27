---
title: error
---

type error
==========

Predeclared `error` type:

    type error interface {
        Error() string
    }

package errors
==============

    package errors

    type errorString struct { text string }

    func (e *errorString) Error() string { return e.text }

    func New(text string) error {
        return &errorString{text}
    }
