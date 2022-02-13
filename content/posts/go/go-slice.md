---
title: "Go Slice"
date: 2022-02-09T13:22:09+08:00
draft: true
toc: true
tags: 
  - go
---

## init

- var s []int **nil 不需要内存分配 **
- s := []int{} **空 slice**
- s := []int{5}
- s := make([]int , 2 , 3)
- from slice/arr [)
- s := *new([]int
- )

```
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

> https://asciiflow.com/#/share/eJy9kUEOgjAQRa8ymTULqSJIwkmoi4Z0YYLEIAsI4RbGhcdwzWm8gFfwCwqoaEATm%2F%2BSGTL9%2FS05R2qt2eVtuAo0GxyqTMfoc8mpZHfhCENyhko4FqpEpwkaydRZKo5VRufy4JFvTparKJEygrozlT%2B5Xj3sW669JAr181jvOu3KWhhu6g%2FqN9mXV2qLe57Ku%2Beg7zQ6HpoJMIEAUzADFpgDGzhg0Xjf3%2FpN1D%2Ff4eWpT%2Ftjp7xFxk8mj0Q7%2Frqt%2FfBzggGXo0BtkMh6SDRK2AKPgUElF1xcAC4tva4%3D)

![slice-from-array](https://s2.loli.net/2022/02/12/x9yQYwsobrBUgav.png)



## 扩容

扩容只关心容量，会把原 slice 的数据拷贝到新 silce 中，**追加数据由 append 在扩容后完成**。

- 原 slice 容量小于 1024，新 slice 容量将扩大为原来的 2 倍
- 原 slice 容量大于或等于 1024，新 slice 容量将扩大为原来的 1.25 倍

## copy

将 src 的数据逐个拷贝到 dst 指向的数组中，数量二者最小值，拷贝过程不会发生扩容。

## express

- 简单表达式 a[low : high] 				
- 拓展表达式 a[low : high : max]      
- 简单表达式操作字符串或数组 0<= low <=high <= len(a)
- **简单表达式操作切片  0<= low <= high <=cap(a)**
- **操作字符串 -> 产生新的字符串**

**s[low:high]  默认值分别为 0，len(s)**

## max

**拓展表达式限制写覆盖，low 可以省略，只能用于数组和切片，不能用于字符串**

a[low:high:max]，max-low 限制新切片的 cap。

```go
package main

import "fmt"

func main() {

	a := [5]int{1, 2, 3, 4, 5}
	b := a[1:4]
	fmt.Printf("a:%v\n", a)
	b = append(b, 0)

	c := a[1:3:3]
	fmt.Printf("c addr:%p\n", c)
	fmt.Printf("c:%v\n", c)
	c = append(c, 0)
	fmt.Printf("c addr:%p\n", c)
	fmt.Printf("a:%v\n", a)
	fmt.Printf("b:%v\n", b)
	fmt.Printf("c:%v\n", c)
}
//
a:[1 2 3 4 5]
c addr:0xc00012c038
c:[2 3]
c addr:0xc000122040
a:[1 2 3 4 0]
b:[2 3 4 0]
c:[2 3 0]

```



> https://asciiflow.com/#/share/eJyrVspLzE1VslIqzslMTtVNrSgoSi0uVtJRykmsTC0CilfHKFXEKFlZWpjpxChVAllGFuZAVklqRQmQE6OkgB3kpOYp2CokJxYo2BrFxOThUAUBj6bsgSCgQjgbF8KqXQGuWUEB7BGIKKbSaXtAmAR7CKrDhWDOSSwqSqzEMAdisgEQGwKxERAbA7EJEJsCsRkQmwOxBfFhAicpci0k1owUkM1EDr1NJLkGJYqgNoBTBLINpLqRgBqlWqVaADqdeVA%3D)

![slice-express](https://s2.loli.net/2022/02/12/W23dNa7ClsShxVc.png)



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

