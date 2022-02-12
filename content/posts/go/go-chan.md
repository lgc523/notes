---
title: "Go Chan"
date: 2022-02-09T13:22:06+08:00
draft: true
toc: true
tags: 
  - go
---

FIFO ，数据总是按照写入的顺序流出管道。

## 声明和初始化

```go
var ch chan int 
ch := make(chan int)
ch := make(chan int, 1)
```

## 管道操作

操作符 <- , -> 表示数据流向，默认双向可读写，在函数间传递时可以使用操作符限制读写。

### 没有缓存区

读会阻塞，直到有 G 写数据

写会阻塞，知道有 G 读数据

### 有缓存区无数据

读会阻塞，知道有 G 写数据

### 有缓冲区满了

写会阻塞，直到有 G 读

### nil

读写永久阻塞

### close

close(ch)，关闭 chan，关闭的 chan 写操作会 panic，关闭后仍然可读

### 读表达式

```go
val := <- ch
val, ok := <-ch
```

**第二个变量表示是否读取成功**

- chan 关闭，缓冲区没数据
  - 读取返回类型零值，false
- chan 关闭，缓冲区有数据
  - 读取 chan 数据，true
  - chan 关闭，并且缓冲区没有数据，false = 管道的关闭状态

## hchan

``runtime/chan.go``

``队列`` ``类型信息`` ``协程等待队列``

```
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16 //每个元素的大小
	closed   uint32 //标识关闭状态
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```

### 环形队列

> https://asciiflow.com/#/share/eJyrVspLzE1VssorzcnRUcpJrEwtUrJSqo5RqohRsjI0MDTRiVGqBDKNLMyBrJLUihIgJ0bp0ZQ9ZKCYmDwgqYAGUCUykjMS8%2BASGCpphyBuIANhuJF4Cu7vwuT80rwSBVsFI1R%2Fo1g1bRdYygCiwhCFMsBCUewjGJMO4a6gkJJYklhYnFkFDAQz9DCgmk9g3GmbUDjICTOpNI2o5IYl0qnnXmxJCs10EKc4NS%2BlAhhgxrSOJLTYoqLPEJ4pSk0uA3nGkFqeIdutMUq1SrUA6MnbaQ%3D%3D)

![make-chan](https://s2.loli.net/2022/02/12/hxWgS9J8kubTf1X.png)

- dataqsiz  表示队列可缓存 6 个元素
- buf 指向队列的内存
- qcount 表示缓冲区还有两个元素
- sendx 表示下一次写入的位置，[0，6)
- recvx 表示读取位置，[0，6)

数组实现队列比较简单， sendx 和 recvx 相当于队尾和队首。

### 等待队列

![chan-nobuf](https://s2.loli.net/2022/02/12/WrOtNaCophBHSm5.png)

- 读 chan ，buf 为空 或 没有 buf，G 会被阻塞，加入 recvq
- 写 chan，buf 已满 或 没有 buf，G 会被阻塞，加入 sendq 
- 处于 队列中的 G 会被其他 G 的操作唤醒

## Chan operator

### write

- chan buf 有位置，写入 buf 
- chan buf 没位置， G -> sendq，sleep 等待 G read 唤醒
- receq 非空时，表示 buf 没数据，有 G 在等待读，会直接将数据传递给 receq 中的第一个协程，不再写入 buf

### read

- chan buf 有数据，读 buf
- chan buf 无数据，G -> receq ，sleep 等待 G write 唤醒
- sendq 非空，无 buf，直接从 sendq 第一个 G 获取数据

### close

关闭 chan 会唤醒 recess 中所有 G，G 获取到对应类型的零值，同时会唤醒 sendq 中的 G，sendq 中的 G 会 panic。

``panic operator``

- 关闭 nil 的 chan
- 再次关闭
- 关闭后写操作

### select-chan

```go
import (
	"fmt"
	"time"
)

func send(ch chan int, e int) {
	for {
		ch <- e
		time.Sleep(time.Second)
	}
}

func main() {

	ch1 := make(chan int, 10)
	ch2 := make(chan int, 10)

	go send(ch1, 1)
	go send(ch2, 2)

	for {
		select {
		case e := <-ch1:
			fmt.Printf("rece from ch1:%d\n", e)
		case e := <-ch2:
			fmt.Printf("rece from ch2:%d\n", e)
		default:
			fmt.Printf("no element ffrom ch1 and ch2.\n")
			time.Sleep(time.Second)
		}
	}
}
```

### range

```go
import "fmt"

func main() {

	ch := make(chan int, 3)
	ch <- 2
	ch <- 3
	close(ch)
	for e := range ch {
		fmt.Printf("get element from ch:%d\n", e)
	}

}
```

