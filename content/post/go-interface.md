---
title: "Go语言-Interface"
date: 2020-01-17T10:50:01+08:00
draft: false
---

### Go语言interface类型

***interface是go语言中定义的一种类型***

interface的声明方式

``` go
//声明一个接口，定义形状，定义计算面积的方法
type shape interface {
	area() float64
}
```

interface的继承

``` go
//声明一个结构体，定义一种特定的形状square
type square struct {
	side float64
}

// another shape
type circle struct {
	radius float64
}

//继承shape接口的area方法
func (s square) area() float64 {
	return s.side * s.side
}

// which implements the shape interface
func (c circle) area() float64 {
	return math.Pi * c.radius * c.radius
}
```

interface的使用

```go
// 定义一个方法，可传入一种shape
func info(z shape) {
	fmt.Println(z)
	fmt.Println(z.area()) //使用对应的shape的area实现
}

```

完整的程序

```go
package main

import (
	"fmt"
	"math"
)

type square struct {
	side float64
}

// another shape
type circle struct {
	radius float64
}

type shape interface {
	area() float64
}

func (s square) area() float64 {
	return s.side * s.side
}

// which implements the shape interface
func (c circle) area() float64 {
	return math.Pi * c.radius * c.radius
}

func info(z shape) {
	fmt.Println(z)
	fmt.Println(z.area())
}

func main() {
	s := square{10}
	c := circle{5}
	info(s)
	info(c)
}

```

空interface

```go
// 定义函数的参数为一个空接口,可以接收任何类型的参数
func specs(a interface{}) {
	fmt.Println(a)
}
```

[代码参考链接](https://github.com/GoesToEleven/GolangTraining/tree/master/21_interfaces)









