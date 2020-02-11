---
title: Go(一)
date: 2019-09-26 15:41:27
tags:
	Go
categories:
	学习笔记
---

#### 1.1命名

Go中函数名字的开头字母大小决定了该函数在包外的可见性：大写字母开头，这个函数在包外可以使用；小写字母开头不可以在包外部访问。Go语言程序员推荐使用 **驼峰式** 命名。

#### 1.2声明

var、const、type、func：变量、常量、类型、函数实体对象声明    

包声明语句之后是import语句导入依赖的其它包，然后是包一级的类型、变量、常量、函数的声明语句。在包一级声明的名字可以在整个包和引用了该包的源文件中访问。

#### 1.3变量

```go
var 变量名字 类型 = 表达式
```

go语言中所有变量定义时如果未赋值，都会初始化为一个默认值。 

在函数内部，有一种称为简短变量声明语句的形式可用于声明和初始化局部变量。它以“**名字 := 表达式**”形式声明变量，变量的类型根据表达式来自动推导。

```go
freq := rand.Float64() * 3.0
t := 0.0
i, j := 0, 1  //变量声明语句
i, j = j, i // 交换 i 和 j 的值，赋值语句（元组赋值）
```

简短变量声明语句中必须至少要声明一个新的变量，否则不能编译通过。

#### 1.4指针

如果指针名字为p，那么可以说“p指针指向变量x”，或者说“p指针保存了x变量的内存地址”。同时`*p`表达式对应p指针指向的变量的值。一般`*p`表达式读取指针指向的变量的值，这里为int类型的值，同时因为`*p`对应一个变量，所以该表达式也可以出现在赋值语句的左边，表示更新指针所指向的变量的值。

```go
x := 1
p := &x         // p, of type *int, points to x
fmt.Println(*p) // "1"
*p = 2          // equivalent to x = 2
fmt.Println(x)  // "2"
*p++            // x=3,p仍然指向x
```

指针的零值为**nil**，在Go语言中，返回函数中局部变量的地址也是安全的。

```go
var p = f()

func f() *int {
    v := 1
    return &v
}
```

#### 1.5 new 函数

表达式new(T)将创建一个T类型的匿名变量，初始化为T类型的零值，然后返回变量地址，返回的指针类型为`*T`。

```go
p := new(int)  // p, *int 类型, 指向匿名的 int 变量
fmt.Println(*p) // "0"
//下面两个函数功能相同
func newInt() *int {
    return new(int)
}

func newInt() *int {
    var dummy int
    return &dummy
}
```

由于new只是一个预定义的函数，它并不是一个关键字，因此我们可以将new名字重新定义为别的类型。下面由于new被定义为int类型的变量名，因此在delta函数内部是无法使用内置的new函数的。

```go
func delta(old, new int) int { return new - old }
```

了解go语言自动垃圾回收器。

#### 1.6 类型

```go
type 类型名字 底层类型

type Celsius float64    // 摄氏温度
```

1.7下划线 ‘_’

```go
//用在import
//引入包，会先调用包中的初始化函数，这种使用方式仅让导入的包做初始化，而不使用包中其他功能
import  _  "net/http/pprof"

//用在返回值
//表示忽略某个值。单函数有多个返回值，用来获取某个特定的值
for _,v := range Slice{}
_,err := func()

//用在变量
//上面用来判断 type T是否实现了I,用作类型断言，如果T没有实现借口I，则编译错误.
type T struct{}
var _ I = T{}

其中 I为interface

//用在函数定义中省略带名的参数
func demo() ( int, int, string, int ,error)
//返回多个参数时,尤其是类型相同的，不利于阅读，

func demo() ( sumA int, sumB int, _ string, total int ,_ error)
//返回参数 int 加了名字，对比上面,便于理解,但是 "string" 和 "error"没有名字，编译会报错，用下划线可以忽略命名
```

#### 1.8 **os.Args**

由os包实现，用于给出程序运行时的参数。os.Args的类型是 **[]string** ，也就是字符串切片。所以可以在for循环的range中遍历，还可以用 **len(os.Args)** 来获取其数量。

```go
package main

import (
    "fmt"
    "os"
    "strconv"
)

func main () {
    for idx, args := range os.Args {
        fmt.Println("参数" + strconv.Itoa(idx) + ":", args)
    }
}
//运行结果
$go run main.go 1 3 -X ?
参数0: /tmp/go-build116558042/command-line-arguments/_obj/exe/main
参数1: 1
参数2: 3
参数3: -X
参数4: ?
//参数0为程序路径本身

```

#### 1.9 **flag**包

使用flag包，首先定义待解析命令行参数，也就是以”-“开头的参数，比如这里的 -b -s -help等。-help不需要特别指定，可以自动处理。      

flag使用前，必须首先解析：`flag.Parse()` 。  参数中没有能够按照预定义的参数解析的部分，通过flag.Args()即可获取，是一个字符串切片。        

通过flag.Bool和flag.String，建立了2个指针b和s，分别指向bool类型和string类型的变量。所以后续要通过 ***b** 和 ***s** 使用变量值。

```go
package main
import (
    "fmt"
    "flag"
)
//参数：命令行参数名称，默认值，提示字符串
var b = flag.Bool("b", false, "bool类型参数")
var s = flag.String("s", "", "string类型参数")

func main() {
    flag.Parse()
    fmt.Println("-b:", *b)
    fmt.Println("-s:", *s)
    fmt.Println("其他参数：", flag.Args())
}
------------------------------------
$ go run main.go
-b: false
-s: 
其他参数： []
------------------------------------
$ go run main.go -b
-b: true
-s: 
其他参数： []
------------------------------------
$ go run main.go -b -s test others
-b: true
-s: test
其他参数： [others]
------------------------------------
$ go run main.go  -help
Usage of /tmp/go-build080353851/command-line-arguments/_obj/exe/main:
  -b    bool类型参数
  -s string
        string类型参数
exit status 2

```

#### 1.10 **err**

error类型是一个接口类型，定义如下：

```go
type error interface {
    Error() string
}

```

可以通过实现error接口来生成错误信息，如果产生错误，就会生成一个non-nil的error对象，将此对象与nil比较，结果为true。

```go
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math: square root of negative number")
    }
    // 实现
}

result, err:= Sqrt(-1)

if err != nil {
   fmt.Println(err)
}

```

#### 1.11 包的初始化和标准输入输出

`func init() { /* ... */ }`

初始化函数在包调用时自动执行。

```go
func main() {
    input := bufio.NewScanner(os.Stdin)//初始化一个扫表对象
    for input.Scan() {//扫描输入内容
        line := input.Text()//把输入内容转换为字符串
        fmt.Println(line)//输出到标准输出
    }
}

```

#### 1.12 作用域

if和switch语句也会在条件部分创建隐式词法域，还有它们对应的执行体词法域。

```go
if x := f(); x == 0 {
    fmt.Println(x)
} else if y := g(x); x == y {
    fmt.Println(x, y)
} else {
    fmt.Println(x, y)
}
fmt.Println(x, y) // compile error: x and y are not visible here

```

在包级别，声明的顺序并不会影响作用域范围，因此一个先声明的可以引用它自身或者是引用后面的一个声明，这可以让我们定义一些相互嵌套或递归的类型或函数。但是如果一个变量或常量递归引用了自身，则会产生编译错误。

```go
var cwd string
//使用声明语句，初始化的cwd为局部变量，导致全局变量cwd并未被初始化
func init() {
    cwd, err := os.Getwd() // NOTE: wrong!
    if err != nil {
        log.Fatalf("os.Getwd failed: %v", err)
    }
    log.Printf("Working directory = %s", cwd)
}

```

```go
var cwd string
//将err先定义，再使用赋值方法
func init() {
    var err error
    cwd, err = os.Getwd()
    if err != nil {
        log.Fatalf("os.Getwd failed: %v", err)
    }
}

```

