---
title: Goroutines
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

        for a := range chA { // consume one
            fmt.Println("A:", a)
            time.Sleep(time.Second * 1)
            go func(a int) {
                chA <- a + 1 // produce one
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

        for a := range chA { // consume one
            fmt.Println("A:", a)
            for i := 0; i < 10; i++ { // produce many
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
            for a := range chA { // consume one
                time.Sleep(time.Second * 1)
                fmt.Println("A:", a)
                chB <- a + 1 // produce one
            }
        }()

        for b := range chB { // consume one
            fmt.Println("B:", b)
            chA <- b + 1 // produce one
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
                go func(a int) { // goroutines accumulated
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
                for a := range chA {
                    time.Sleep(time.Second * 1) // simulate real work
                    fmt.Println("A:", a)
                    go func(a int) { // goroutine accumulation
                        chB <- a
                    }(a)
                }
            }()
        }

        for b := range chB { // consume one
            fmt.Println("B:", b)
            for i := 0; i < 10; i++ { // produce many
                chA <- i
            }
        }
    }

### One-to-many with counting semaphore

    func main() {
        worklist := make(chan []string)

        go func() { worklist <- os.Args[1:] }()

        seed := make(map[string]bool)
        for list := range worklist { // consume one
            for _, link := range list {
                if !seen[link] {
                    seed[link] = true
                    go func(link string) { // goroutine accumulation
                        worklist <- crawl(link) // produce many
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
