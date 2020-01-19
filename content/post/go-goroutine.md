---
title: "Go语言-Goroutine和Channel"
date: 2020-01-19T10:18:29+08:00
draft: false
tags: ["go", "goroutine","coding"]
categories: ["go"]
---

***goroutine是go语言的核心概念，即go协程，是go语言实现并发编程的基础。***

### 定义和启动一个go协程

```go
package main
import "fmt"

func main() {
	//go关键字启动一个协程
  go foo()
	go bar()
}
/*
定义函数 
go中定义的每一个函数都可以以协程的形式执行
*/
func foo() {
	for i := 0; i < 45; i++ {
		fmt.Println("Foo:", i)
	}
}

func bar() {
	for i := 0; i < 45; i++ {
		fmt.Println("Bar:", i)
	}
}
```



### WaitGroup

来自sync.WaitGroup包，可以用来控制和维护子协程，保证主协程在子协程执行结束之后退出。它能够一直等到所有的goroutine执行完成，并且阻塞主线程的执行，直到所有的goroutine执行完成。

```go
var wg sync.WaitGroup

func main() {
  //waitgroup中增加计数
	wg.Add(2)
	go foo()
	go bar()
  //阻塞主线程，等待waitgroup中的计数归0，退出主线程
	wg.Wait()
}

func foo() {
	for i := 0; i < 45; i++ {
		fmt.Println("Foo:", i)
	}
  //waitgroup中的计数减1
	wg.Done()
}

func bar() {
	for i := 0; i < 45; i++ {
		fmt.Println("Bar:", i)
	}
	wg.Done()
}
```

### time.Sleep

```go
func foo() {
	for i := 0; i < 45; i++ {
		fmt.Println("Foo:", i)
    //sleep
		time.Sleep(3 * time.Millisecond)
	}
	wg.Done()
}
```

### go run -race

golang检测race condition的工具：[Data Race Detector](https://golang.org/doc/articles/race_detector.html)；使用方式：go run -race main.go

### 解决race condition的***Mutex互斥锁***

```go
var wg sync.WaitGroup
var counter int
var mutex sync.Mutex

func main() {
	wg.Add(2)
	go incrementor("Foo:")
	go incrementor("Bar:")
	wg.Wait()
	fmt.Println("Final Counter:", counter)
}

func incrementor(s string) {
	for i := 0; i < 20; i++ {
		time.Sleep(time.Duration(rand.Intn(20)) * time.Millisecond)
		//mutex为代码区间加锁
    mutex.Lock()
		counter++
		fmt.Println(s, i, "Counter:", counter)
		mutex.Unlock()
	}
	wg.Done()
}
```



### 解决race condition的***atomic***


```go
import (
	"fmt"
	"math/rand"
	"sync"
	"sync/atomic"
	"time"
)

var wg sync.WaitGroup
var counter int64s

func main() {
	wg.Add(2)
	go incrementor("Foo:")
	go incrementor("Bar:")
	wg.Wait()
	fmt.Println("Final Counter:", counter)
}

func incrementor(s string) {
	for i := 0; i < 20; i++ {
		time.Sleep(time.Duration(rand.Intn(3)) * time.Millisecond)
		//原子加法
    atomic.AddInt64(&counter, 1)
		fmt.Println(s, i, "Counter:", atomic.LoadInt64(&counter)) // access without race
	}
	wg.Done()
}
```



### channel

***unbuffered channel***

```go 
import (
	"fmt"
	"time"
)

func main() {
	//在主协程中定义一个 int类型的无缓冲通道
  c := make(chan int)

  //生产方法进行channel的写入
	go func() {
		for i := 0; i < 10; i++ {
			c <- i
		}
	}()
	//消费方法进行消息的读取
	go func() {
		for {
			fmt.Println(<-c)
		}
	}()

	time.Sleep(time.Second)
}

```

***buffered channel*** 





***range***消费channel

```go
import (
	"fmt"
)

func main() {
	c := make(chan int)

	go func() {
		for i := 0; i < 10; i++ {
			c <- i
		}
		close(c)
	}()

	for n := range c {
		fmt.Println(n)
	}
}
```



***semaphore***

Semaphore和waitgroup具有相似的能力，使用一个channel记录信号量，分别在生产协程和消费协程中写入和读取channel的数据，可以使主协程在子协程完成之后退出。

疑问：为什么这里主协程会阻塞直到子协程执行完才退出呢？

```go
package main

import (
	"fmt"
)

func main() {

	c := make(chan int)
  //定义第二个通道作为信号量标记
	done := make(chan bool)

	go func() {
		for i := 0; i < 10; i++ {
			c <- i
		}
    //子协程中写入通道，作为信号量标记
		done <- true
	}()

	go func() {
		for i := 0; i < 10; i++ {
			c <- i
		}
    //子协程中写入通道，作为信号量标记
		done <- true
	}()

	go func() {
    //子协程中消费信号量
		<-done
		<-done
		close(c)
	}()

	for n := range c {
		fmt.Println(n)
	}
}

```

***channel作为函数返回值***

```go
import "fmt"

func main() {
	c := incrementor()
	cSum := puller(c)
	for n := range cSum {
		fmt.Println(n)
	}
}

func incrementor() chan int {
	out := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			out <- i
		}
		close(out)
	}()
	return out
}

func puller(c chan int) chan int {
	out := make(chan int)
	go func() {
		var sum int
		for n := range c {
			sum += n
		}
		out <- sum
		close(out)
	}()
	return out
}
```



***channel方向***

***chan <-*** 表示只接受生产的通道

***<- chan*** 表示只接受消费的通道



[代码参考链接](https://github.com/GoesToEleven/GolangTraining/tree/master/22_go-routines)

