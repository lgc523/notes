---
title: "Go Slice"
date: 2022-02-09T13:22:09+08:00
draft: true
toc: true
tags: 
  - go
---

> https://go.dev/play/p/iKtfAiv4upU

```go
func main() {
	a := []int{1, 2, 3}
	b := a[:2:2]
	fmt.Printf("addr a: %p, b: %p\n", a, b)
	fmt.Printf("b len: %d,cap: %d\n", len(b), cap(b))
	fmt.Printf("slice b %v\n", b)
	b = append(b, 5)
	fmt.Printf("slice b %v\n", b)
	fmt.Printf("a len: %d,cap: %d\n", len(a), cap(a))
	fmt.Printf("b len: %d,cap: %d\n", len(b), cap(b))
	fmt.Printf("addr a: %p, b: %p\n", a, b)

}
```

```
addr a: 0xc0000b8000, b: 0xc0000b8000
b len: 2,cap: 2
slice b [1 2]
slice b [1 2 5]
a len: 3,cap: 3
b len: 3,cap: 4
addr a: 0xc0000b8000, b: 0xc0000bc000

```

