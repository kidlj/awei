---
title: Slice
---

### index

slice 允许 index 到 len(s) 而不报错：

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

### make

make() 如果 len 参数不为 0，返回的 slice 是有零值填充的。

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

### append

append() 总是会返回一个新的 slice，length 和原来的不同。

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

### modify

修改 slice 元素会修改 underlying array，而且不会返回一个新的 slice。

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

### reverse

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

### rotate

    func ExampleRotate() {
        // Roteate s left by two positions.
        s := []int{0, 1, 2, 3, 4}
        reverse(s[:2])
        reverse(s[2:])
        reverse(s)
        fmt.Println(s)
        // Output:
        // [2 3 4 0 1]
    }

