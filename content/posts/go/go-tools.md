---
title: "Go Tools"
date: 2022-02-14T08:27:01+08:00
draft: true
toc: true
tags: 
  - go
---

## go tool nm 

```
go tool compile -l -p main main.go -> OBJ
go tool nm main.o 方法列表
```

```go
package main

import "fmt"

type People struct {
}

func (p *People) ShowA() {
	fmt.Println("showA")
	p.ShowB()
}

func (p *People) ShowB() {
	fmt.Println("ShowB")
}

type Teacher struct {
	People
}

func (t *Teacher) ShowB() {
	fmt.Println("teacher showB")
}

func main() {
	t := Teacher{}
	t.ShowA()

}
```



```go
package main

type A int

func (a A) Value() int {

	return int(a)
}

func (a *A) Set(n int) {
	*a = A(n)
}

type B struct {
	A
	b int
}

type C struct {
	*A
	c int
}
```



```go
    454d T structA.(*A).Set
    52b7 T structA.(*A).Value
    5319 T structA.(*B).Set
    537b T structA.(*B).Value
    543d T structA.(*C).Set
    549f T structA.(*C).Value
    454c T structA.A.Value
    53d1 T structA.B.Value
    54fc T structA.C.Set
    5575 T structA.C.Value
```

