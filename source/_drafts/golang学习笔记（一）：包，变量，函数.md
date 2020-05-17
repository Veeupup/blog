---
title: golang学习笔记（一）：包，变量，函数
date: 2020-02-14 23:25:09
---

> 欢迎访问我的[博客](http://blog.tanweime.com/)和[github](https://github.com/veeupup)!

go 语言学习笔记，来自 [gotour](https://tour.golang.org/list) ，以后要常写笔记，把自己学习笔记记录下来，就算只是笔记也要多写。

好记性不如烂笔头，也要多锻炼自己的写作能力。

说实话，今天很累了，最近在折腾操作系统内核，因为原先写了个bootloader，现在想要转向 grub 来，遇到坑太多了，已经两天了😭。

还是接触一点新知识简单的东西，来缓冲一下，脑子迷迷糊糊的。

<!--more-->

# **package**

每个Go程序由很多包组成。

程序都是从 main 包开始运行。

该程序正在使用导入路径为“ fmt”和“ math / rand”的软件包。

按照约定，程序包名称与导入路径的最后一个元素相同。

 例如，“ math / rand”包包括以语句包rand开头的文件。

## import

此代码将导入分组为带括号的“分解的”导入语句。

您还可以编写多个导入语句，例如：

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	fmt.Println(math.Pi)
}

```

但是使用分解式import语句是一种很好的样式。

## **导出名称**

在Go中，如果名称以大写字母开头，则导出该名称。

例如，Pizza是一个导出的名称，Pi也是，它是从math包导出的。

pizza和pi不以大写字母开头，所以它们不被导出。

在导入包时，您只能引用它导出的名称。任何“未导出”的名称都不能从包外部访问。

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	fmt.Println(math.Pi)
}

```

# **函数**

一个函数可以接受零个或多个参数。

在此示例中，add接受两个类型为int的参数。

请注意，类型位于变量名称之后。

```go
package main

import "fmt"

func add(x int, y int) int {
	return x + y
}

func mins(x int, y int) int {
	return x - y;
}

func main() {
	fmt.Println(add(42, 13))
	fmt.Println(mins(23, 11))
}

```

当两个或多个连续的命名函数参数共享一个类型时，可以从除最后一个之外的所有其他参数中省略该类型。

在这个例子中，我们缩短了

```go
x int，y int
```

至

```go
x，y int

```

```go
package main

import "fmt"

func add(x, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}

```

## 多个返回值

一个函数能返回多个返回值。

```go
package main

import "fmt"

func swap(a, b string) (string, string) {
	return b, a
}

func main() {
	a, b := swap("ai", "ni")
	fmt.Println(a, b)
}

```

## Named return values

Go的返回值可能会被命名。

如果是，则将它们视为定义在函数顶部的变量。

应该使用这些名称来记录返回值的含义。

没有参数的return语句返回指定的返回值。

这就是所谓的“naked” return。

裸返回语句应该只在短函数中使用，如下面的示例所示。

在较长的函数中，它们可能会损害可读性。

```go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}


func main() {
	fmt.Println(split(17))
}

```



# **变量**

var语句声明了一个变量列表;在函数参数列表中，类型是最后一个。

var语句可以是包级的，也可以是函数级的。在这个例子中我们可以看到两者。

```go
package main

import "fmt"

var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}

```

## 初始化数值

var声明可以包含初始化器，每个变量一个。

如果有初始化，类型可以省略;该变量将采用初始化器的类型。

```go
package main

import "fmt"

var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}

```

## **短变量初始化**

在函数内部，可以使用:= short赋值语句来代替具有隐式类型的var声明。

在函数之外，每个语句都以一个关键字(var、func等)开头，因此:=结构不可用

```go
package main

import "fmt"

func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"
	fmt.Println(a ,i, j, k, c, python, java)
}

```

## 基本数据类型

go 语言的基本数据类型

```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```

该示例显示了几种类型的变量，并且与import语句一样，变量声明也可以“分解”为块。

int，uint和uintptr类型通常在32位系统上为32位宽，在64位系统上为64位宽。

 当您需要整数值时，应该使用int，除非有特殊原因要使用大小或无符号整数类型。

```go
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)



func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}

```

## **零值**

声明时没有明确的初始值的变量将被赋予零值。

零值为：

数字类型为0，

对于布尔类型为false

“”（空字符串）表示字符串。

```go
package main

import "fmt"

func main() {
	var i int
	var f float64
	var b bool
	var s string
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
}

```

## **类型转换**

使用 T(v) 将 v 值转换为 T 类型

一些例子：

```go
package main

import "fmt"

func main() {
	v := 12.2 // change me!
	fmt.Printf("v is of type %T\n", v)
}

```

与C语言不同，在Go语言中，不同类型的项目之间的分配需要显式转换。

## **类型推断**

在声明变量而不指定显式类型时（使用：=语法或var =表达式语法），将从右侧的值推断出变量的类型。

键入声明的右侧时，新变量具有相同的类型：

```go
var i int j：= i // j是一个整数
```

但是，当右侧包含无类型的数字常量时，新变量可能是int，float64或complex128，具体取决于常量的精度：

```go
i：= 42 //整数 

f：= 3.142 // 

float64 g：= 0.867 + 0.5i //complex128
```

```go
package main

import "fmt"

func main() {
	v := 12.2 // change me!
	fmt.Printf("v is of type %T\n", v)
}

```

## **常量**

常量像变量一样声明，但是使用const关键字。

常量可以是字符，字符串，布尔值或数字值。

不能使用：=语法声明常量。

```go
package main

import "fmt"

const Pi = 3.14

func main() {
	const World = "世界"
	fmt.Println("Hello", World)
	fmt.Println("Happy", Pi, "Day")

	const Truth = true
	fmt.Println("Go rules?", Truth)
}

```

## **数字常量**

数字常数是高精度值。

未说明类型的常量采用其上下文所需的类型。

（一个int最多可以存储一个64位整数，有时更少。）

```go
package main

import "fmt"

const (
	// 1 左移 100 位
	Big = 1 << 100
	// 右移 99 位
	Small = Big >> 99
)

func needInt(x int) int { return x*10 + 1 }
func needFloat(x float64) float64 {
	return x * 0.1
}

func main() {
	fmt.Println(needInt(Small))
	fmt.Println(needFloat(Small))
	fmt.Println(needFloat(Big))
}

```

