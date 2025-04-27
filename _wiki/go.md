---
title: Go
---

### Installation

	$ go get golang.org/dl/go1.17.3
	$ ~/go/bin/go1.17.3 download
	$ ln -s ~/go/bin/go1.17.3 ~/go/bin/go
	$ go env

### Slice

#### index

slice 允许 index 到 len(s) 而不报错：

```go
func ExampleIndex() {
	a := []int{}
	b := []int{0}
	var c []int
	fmt.Println(a == nil)
	fmt.Println(c == nil)
	fmt.Println(a[0:])
	fmt.Println(b[1:])
	fmt.Println(c[0:])
	fmt.Println(a[1:]) // panic: runtime error: slice bounds out of range
	// Output:
	// false
	// true
	// []
	// []
	// []
}
```

#### make

make() 如果 len 参数不为 0，返回的 slice 是有零值填充的。

```go
func ExampleMake() {
	a := make([]int, 4)
	b := make([]int, 0, 4)
	a = append(a, 1)
	b = append(b, 1)
	fmt.Println(a)
	fmt.Println(b)
	// Output:
	// [0 0 0 0 1]
	// [1]
}
```

#### append

append() 总是会返回一个新的 slice，length 和原来的不同。

```go
func ExampleAppend() {
	a := make([]int, 0, 10)
	b := a
	a = append(a, 1)
	fmt.Println(a)
	fmt.Println(b)
	// Wrong Output:
	// [1]
	// [1] ✗

	// Output:
	// [1]
	// []

	// Javascript array:
	// a = [1, 2, 3]
	// b = a
	// a.push(4)
	// console.log(a)
	// console.log(b)

	// Javascript Output:
	// [1, 2, 3, 4]
	// [1, 2, 3, 4]
}
```

#### modify

修改 slice 元素会修改 underlying array，而且不会返回一个新的 slice。

```go
func ExampleModify() {
	a := []int{1, 2, 3}
	b := a
	fmt.Println(a)
	fmt.Println(b)
	a[0] = 0
	fmt.Println(a)
	fmt.Println(b)
	// Output:
	// [1 2 3]
	// [1 2 3]
	// [0 2 3]
	// [0 2 3]
}
```

#### reverse

```go
func reverse(s []int) {
	for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
		s[i], s[j] = s[j], s[i]
	}
}

func ExampleReverse() {
	s := []int{0, 1, 2, 3, 4}
	reverse(s)
	fmt.Println(s)
	// Output:
	// [4 3 2 1 0]
}
```

#### rotate

```go
func ExampleRotate() {
	// Rotate s left by two positions.
	s := []int{0, 1, 2, 3, 4}
	reverse(s[:2])
	reverse(s[2:])
	reverse(s)
	fmt.Println(s)
	// Output:
	// [2 3 4 0 1]
}
```

### Channels

#### One-to-one dead lock

```go
func main() {
    chA := make(chan int)

    go func() { chA <- 0 }()

    for a := range chA {
        fmt.Println("A:", a)
        time.Sleep(time.Second * 1)
        chA <- a + 1
    }
}
```

#### One-to-one 

```go
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
```

#### One-to-many dead lock

```go
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
```

#### One-to-many

```go
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
```

#### Two channels: One-to-one

```go
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
```

#### Two channels: One-to-many dead lock

```go
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
```


#### Two channels: One-to-many

```go
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
```

#### One-to-many with multiple worker goroutines

```go
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
```

#### One-to-many with counting semaphore

```go
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
```