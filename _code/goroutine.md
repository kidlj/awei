---
title: goroutine
---

One channel
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

        for a := range chA { // consumes one
            fmt.Println("A:", a)
            time.Sleep(time.Second * 1)
            go func(a int) {
                chA <- a + 1 // produces one
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

        for a := range chA { // consumes one
            fmt.Println("A:", a)
            for i := 0; i < 10; i++ { // produces many
                go func(i int) {
                    chA <- i // no sequence guaranteed
                }(i)
            }
        }
    }


Two channels
============

### One-to-one

    func main() {
        chA := make(chan int)
        chB := make(chan int)

        go func() { chB <- 0 }()

        go func() {
            for a := range chA { // consumes one
                time.Sleep(time.Second * 1)
                fmt.Println("A:", a)
                chB <- a + 1 // produces one
            }
        }()

        for b := range chB { // consumes one
            fmt.Println("B:", b)
            chA <- b + 1 // produces one
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
                chB <- a
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

        go func() { // worker goroutine
            for a := range chA {
                time.Sleep(time.Second * 1)
                fmt.Println("A:", a)
                go func(a int) { // 累积 goroutine
                    chB <- a // no sequence guaranteed
                }(a)
            }
        }()

        for b := range chB {
            fmt.Println("B:", b)
            for i := 0; i < 10; i++ {
                chA <- i // sequence guaranteed
            }
        }
    }

### One-to-many with multiple worker goroutines

    func main() {
        chA := make(chan int)
        chB := make(chan int)

        go func() { chB <- 0 }()

        for i := 0; i < 20; i++ {
            go func() {
                for a := range chA { // consumes one
                    time.Sleep(time.Second * 1)
                    fmt.Println("A:", a)
                    go func(a int) {
                        chB <- a // produces one
                    }(a)
                }
            }()
        }

        for b := range chB { // consumes one
            fmt.Println("B:", b)
            for i := 0; i < 10; i++ { // produces many
                chA <- i
            }
        }
    }

### One-to-many with counting semaphore

    func main() {
        worklist := make(chan []string)

        go func() { worklist <- os.Args[1:] }()

        seed := make(map[string]bool)
        for list := range worklist {
            for _, link := range list {
                if !seen[link] {
                    seed[link] = true
                    go func(link string) {
                        worklist <- crawl(link)
                    }(link)
                }
            }
        }
    }

    var tokens = make(chan struct{}, 20)

    func crawl(url string) []string {
        tokens <- struct{}{} // acquire a token
        list, err := links.Extract(url)
        <-tokens // release the token
        if err != nil {
            log.Printf(err)
        }
        return list
    }
