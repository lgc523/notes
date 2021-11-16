---
title: "Golang Programming Language"
date: 2021-08-11T01:17:20+08:00
draft: true
author: "spider"
toc: true
images:
tags:
  - go
---

```go
package main

import (
	"fmt"
	"image"
	"image/color"
	"image/gif"
	"io"
	"log"
	"math"
	"math/rand"
	"net/http"
	"sync"
)

var mu sync.Mutex
var count int

func main() {
	http.HandleFunc("/", handler)
	http.HandleFunc("/count", counter)
	http.HandleFunc("/lsr", func(w http.ResponseWriter, r *http.Request) {
		//handler(w,r)
		lissajous(w)
	})
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "%s\t\t%s\t%s\n", r.Method, r.URL, r.Proto)
	for k, v := range r.Header {
		fmt.Fprintf(w, "Header[%q]=%q\n", k, v)
	}
	fmt.Fprintf(w, "Host = %q\n", r.Host)
	fmt.Fprintf(w, "RemoteAddr = %q\n", r.RemoteAddr)
	if err := r.ParseForm(); err != nil {
		log.Print(err)
	}
	for k, v := range r.Form {
		fmt.Fprintf(w, "Form[%q] = %q\n", k, v)
	}
	mu.Lock()
	count++
	mu.Unlock()
	//fmt.Fprintf(w, "HTTP.METHOD:%s\nURL.PATH=%q\n", r.Method, r.URL.Path)
}

//count
func counter(w http.ResponseWriter, r *http.Request) {
	mu.Lock()
	fmt.Fprintf(w, "Count %d\n", count)
	mu.Unlock()
}

func lissajous(out io.Writer) {
	mu.Lock()
	count++
	mu.Unlock()
	var palette = []color.Color{color.Opaque, color.Black}

	const (
		whiteIndex = 2
		blackIndex = 3
	)
	const (
		cycles  = 5
		res     = 0.001
		size    = 100
		nframes = 64
		delay   = 8
	)
	freq := rand.Float64()
	anim := gif.GIF{LoopCount: nframes}
	phase := 0.0
	for i := 0; i < nframes; i++ {
		rect := image.Rect(0, 0, 2*size+1, 2*size+1)
		img := image.NewPaletted(rect, palette)
		for t := 0.0; t < cycles*2*math.Pi; t += res {
			x := math.Sin(t)
			y := math.Sin(t*freq + phase)
			img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5), blackIndex)
		}
		phase += 0.1
		anim.Delay = append(anim.Delay, delay)
		anim.Image = append(anim.Image, img)
	}
	gif.EncodeAll(out, &anim)

}

```

## printf verb

| Verb     | Desc                                                  |
| -------- | ----------------------------------------------------- |
| %d       | 十进制                                                |
| %x %o %b | 十六进制、八进制、二进制整数                          |
| %f %g %e | 浮点数，g自动保持足够的精度，最简洁，e有指数，f无指数 |
| %t       | 布尔型                                                |
| %c       | 字符 unicode码点                                      |
| %s       | 字符串                                                |
| %q       | 带引号的字符串/字符                                   |
| %v       | 内置格式的任何值                                      |
| %T       | 任何值的类型                                          |
| %%       | 百分号                                                |
| %[1]     | 重复使用第一个操作数                                  |
| %#       | 输出相应的前缀                                        |

## new

New 是一种内置创建变量的函数，new(T) 创建一个未命名的T类型变量，初始化为T类型的零值，并返回其地址*T，和取地址的普通局部变量没有什么不同，只是不需要引入一个虚拟的名字。

## escape

**每一次变量逃逸都需要一次额外的内存分配过程**

```go
package main

var global *int

func main() {
}
func f() {
	var x int
	x = 1
  // f() 返回 global 还可以访问 x ，x 从 f 中逃逸，x 使用堆空间
	global = &x
}

func g() {
	y := new(int)
	*y = 1
}

```

## 多重赋值推演

```go
x, y = y, x
i, j = 2, 3
```

## 基本数据类型

- Int、uint = 运算平台上运算效率最高的值

- uintptr 大小不明确，足以完整存放指针
- Rune = int32 四个字节，适合UTF-8 的变长1-4 个
- 二元操作符优先级
  - * /  % <<  >> & &^ + --- | ^ == != < <= > >= && || 降序

- **取模结果正负号和被除数一致**，-5%3 = -2

- 一元运算符 

  - 整数 +1 = 0+1，-x = 0-x
  - 浮点数和复数 +x =x，-x=x的负数

- 溢出丢弃

  ```
  var u uint8 = 255
  //255 0 1
  fmt.Println(u,u+1,u*u)
  var i int8 = 127
  fmt.Println(i,i+1,i*i)
  //127 -128 1
  ```

- 浮点数

  ```
  	fmt.Println(-5%3)//-2
  	fmt.Println(-5%-3)//-2
  	fmt.Println(+1)//1
  	fmt.Println(-1)//-1
  	fmt.Println(-1.1)//-1.1
  	fmt.Println(+1.1)//1.1
  	var u uint8 = 255
  	//255 0 1
  	fmt.Println(u,u+1,u*u)
  	var i int8 = 127
  	fmt.Println(i,i+1,i*i)
  	//127 -128 1
  	fmt.Println(math.NaN())
  	//NaN
  	var z float64
  	fmt.Println(z,-z,1/z,-1/z,z/z)
  	//0 -0 +Inf -Inf NaN
  	fmt.Println(math.IsNaN(z/z))//true
  ```

- 字符串

  字符串的第i个字节不一定就是第i个字符，因为非ASCII字符的UTF-8码点需要两个或更多的字节。

  ```go
  	s:="hello"
  	t:=s
  	s+=",golang" //重新赋值
  	fmt.Println(s)//hello,golang
  	fmt.Println(t)//hello
  	fmt.Println(s[0])//104
  字符串不可变 = 共用一段底层内存，子串的生成操作开销很低，不会分配新的内存。
  ```

  因为文件都是UTF-8编码格式，字符串会按照UTF-8解读，字符串可以包含Unicode码点。

