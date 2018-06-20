---
title: goroutine
---

One Channel
===========

### One-to-one dead lock

    func main() {
        chA := make(chan int)

        go func() { chA <- 0 }()

        for a := range chA {
            fmt.Println("A:", a)
            time.Sleep(time.Second * 1)
            chA <- a + 1
        }
    }

### One-to-one 

    func main() {
        chA := make(chan int)

        go func() { chA <- 0 }()

        for a := range chA {
            fmt.Println("A:", a)
            time.Sleep(time.Second * 1)
            go func(a int) {
                chA <- a + 1
            }(a)
        }
    }

### One-to-many dead lock

    func main() {
        chA := make(chan int)

        go func() { ChA <- 0 }()

        for a := range chA {
            fmt.Println("A:", a)
            for i := 0; i < 10; i++ {
                chA <- i
            }
        }
    }

### One-to-many

    func main() {
        chA := make(chan int)

        go func() { ChA <- 0 }()

        for a := range chA {
            fmt.Println("A:", a)
            for i := 0; i < 10; i++ {
                go func(i int) {
                    chA <- i // no sequence guaranteed
                }(i)
            }
        }
    }


Two Channels
============

### One-to-one

    func main() {
        chA := make(chan int)
        chB := make(chan int)

        go func() { chB <- 0 }()

        go func() {
            for a := range chA {
                time.Sleep(time.Second * 1)
                fmt.Println("A:", a)
                chB <- a + 1
            }
        }()

        for b := range chB {
            fmt.Println("B:", b)
            chA <- b + 1
        }
    }


### One-to-many dead lock

    func main() {
        chA := make(chan int)
        chB := make(chan int)

        go func() { chB <- 0 }()

        go func() {
            for a := range chA {
                time.Sleep(time.Second * 1)
                fmt.Println("A:", a)
                chB <- a + 1
            }
        }()

        for b := range chB {
            fmt.Println("B:", b)
            for i := 0; i < 10; i++ {
                chA <- i
            }
        }
    }


### One-to-many

    func main() {
        chA := make(chan int)
        chB := make(chan int)

        go func() { chB <- 0 }()

        go func() {
            for a := range chA {
                time.Sleep(time.Second * 1)
                fmt.Println("A:", a)
                chB <- a + 1
            }
        }()

        for b := range chB {
            fmt.Println("B:", b)
            for i := 0; i < 10; i++ {
                go func(i int) {
                    chA <- i // no sequence guaranteed
                }(i)
            }
        }
    }
