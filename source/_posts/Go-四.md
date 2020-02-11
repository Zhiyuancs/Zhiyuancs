---
title: Go(四)
date: 2019-10-01 17:13:30
tags:
	Go
categories:
	学习笔记
---

### 函数

#### 4.1 函数声明

函数声明包括函数名、形式参数列表、返回值列表（可省略）以及函数体。

```go
func name(parameter-list) (result-list) {
    body
}
```

如果一组形参或返回值有相同的类型，我们不必为每个形参都写出参数类型。

```go
func f(i, j, k int, s, t string)                 { /* ... */ }
func f(i int, j int, k int,  s string, t string) { /* ... */ }
```

```go
func add(x int, y int) int   {return x + y}
func sub(x, y int) (z int)   { z = x - y; return}
func first(x int, _ int) int { return x }
func zero(int, int) int      { return 0 }

fmt.Printf("%T\n", add)   // "func(int, int) int"
fmt.Printf("%T\n", sub)   // "func(int, int) int"
fmt.Printf("%T\n", first) // "func(int, int) int"
fmt.Printf("%T\n", zero)  // "func(int, int) int"
```

在函数调用时，Go语言没有默认参数值，也没有任何方法可以通过参数名指定形参，因此形参和返回值的变量名对于函数调用者而言没有意义。

遇到没有函数体的函数声明，这表示该函数不是以Go实现的。这样的声明定义了函数标识符。

```go
func Sin(x float64) float //implemented in assembly language
```

#### 4.2 递归

Go语言使用可变栈，栈的大小按需增加(初始时很小)。这使得我们使用递归时不必考虑溢出和安全问题。

#### 4.3 多返回值

在Go中，一个函数可以返回多个值。我们已经在之前例子中看到，许多标准库中的函数返回2个值，一个是期望得到的返回值，另一个是函数出错时的错误信息。

调用多返回值函数时，返回给调用者的是一组值，调用者必须显式的将这些值分配给变量:

```go
links, err := findLinks(url)
links, _ := findLinks(url) // errors ignored
```

当你调用接受多参数的函数时，可以将一个返回多参数的函数作为该函数的参数。虽然这很少出现在实际生产代码中，但这个特性在debug时很方便，我们只需要一条语句就可以输出所有的返回值。下面的代码是等价的。

```Go
log.Println(findLinks(url))
links, err := findLinks(url)
log.Println(links, err)
```

如果一个函数所有的返回值都显示变量名，那么该函数的return语句可以省略操作数，称之为base return。

```go
func Count(num1,num2 int) (count1,count2 int,err error){
	count1,count2:=num1,num2
    err:=nil
	return 
}
```

#### 4.4 文件结尾错误EOF

io包保证任何由文件结束引起的读取失败都返回同一个错误io.EOF，该错误在io包中定义：

```Go
package io

import "errors"

// EOF is the error returned by Read when no more input is available.
var EOF = errors.New("EOF")
```

调用者只需通过简单的比较，就可以检测出这个错误。

```Go
in := bufio.NewReader(os.Stdin)
for {
    r, _, err := in.ReadRune()
    if err == io.EOF {
        break // finished reading
    }
    if err != nil {
        return fmt.Errorf("read failed:%v", err)
    }
}

```

#### 4.5 函数值

在Go中，函数被看作第一类值（first-class values）：函数像其他值一样，拥有类型，可以被**赋值给其他变量，传递给函数，从函数返回**。对函数值（function value）的调用类似函数调用。

```Go
func square(n int) int { return n * n }
f := square
fmt.Println(f(3)) // "9"

```

函数类型的零值是nil。调用值为nil的函数值会引起panic错误。函数可以和nil比较，但是函数之间是不可以比较的，不可以作为map的key。

函数值作为函数的参数，下面strings.Map对字符串中的每个字符调用add1函数，并将每个add1函数的返回值组成一个新的字符串返回给调用者。

```Go
func add1(r rune) rune { return r + 1 }
//String.Map函数定义
//func Map(mapping func(rune) rune, s string) string
fmt.Println(strings.Map(add1, "HAL-9000")) // "IBM.:111"

```

#### 4.6 匿名函数

函数值字面量是一种表达式，它的值被成为匿名函数（anonymous function）。函数字面量允许我们在使用函数时，再定义它。

```Go
strings.Map(func(r rune) rune { return r + 1 }, "HAL-9000")

```

更为重要的是，通过这种方式定义的函数可以访问完整的词法环境（lexical environment），这意味着在函数中定义的内部函数可以引用该函数的变量：

```Go
func squares() func() int {
    var x int
    return func() int {
        x++
        return x * x
    }
}
func main() {
    f := squares()
    fmt.Println(f()) // "1"
    fmt.Println(f()) // "4"
    fmt.Println(f()) // "9"
    fmt.Println(f()) // "16"
}

```

squares的例子证明，函数值不仅仅是一串代码，还记录了状态，其中的变量生命周期不由它的作用域决定。在squares中定义的匿名内部函数可以访问和更新squares中的局部变量，这意味着匿名函数和squares中，存在变量引用。这就是**函数值属于引用类型**和函数值不可比较的原因。Go使用闭包（closures）技术实现函数值，Go程序员也把函数值叫做闭包。

当匿名函数需要被递归调用时，我们必须首先声明一个变量，再将匿名函数赋值给这个变量。

```Go
visitAll := func(items []string) {
    // ...
    visitAll(m[item]) // compile error: undefined: visitAll
    // ...
}

```

#### 4.7 捕获迭代变量

```go
var rmdirs []func()
for _, dir := range tempDirs() {
    os.MkdirAll(dir, 0755)
    rmdirs = append(rmdirs, func() {
        os.RemoveAll(dir) // NOTE: incorrect!
    })
}
for _, rmdir := range rmdirs {
    rmdir() // clean up
}

```

上面代码是错误的，在该循环中生成的所有函数值都共享相同的循环变量，需要注意，函数值中记录的是循环变量的内存地址，而不是循环变量某一时刻的值。所以后续的迭代会不断更新dir的值，当删除操作执行时，for循环已完成，dir中存储的值等于最后一次迭代的值。

通常，为了解决这个问题，我们会引入一个与循环变量同名的局部变量，作为循环变量的副本。

```Go
for _, dir := range tempDirs() {
    dir := dir // declares inner dir, initialized to outer dir
    // ...
}

```

#### 4.8 可变参数

参数数量可变的函数称为为可变参数函数。在声明可变参数函数时，需要在参数列表的最后一个参数类型之前加上省略符号“...”，这表示该函数会接收任意数量的该类型参数。

```Go
func sum(vals...int) int {
    total := 0
    for _, val := range vals {
        total += val
    }
    return total
}
fmt.Println(sum())           // "0"
fmt.Println(sum(3))          // "3"
fmt.Println(sum(1, 2, 3, 4)) // "10"

```

在上面的代码中，调用者隐式的创建一个数组，并将原始参数复制到数组中，再把数组的一个切片作为参数传给被调函数。如果原始参数已经是切片类型，只需在最后一个参数后加上省略符。

```Go
values := []int{1, 2, 3, 4}
fmt.Println(sum(values...)) // "10"

```

#### 4.9 Deferred函数

在普通函数或方法前加上关键字defer，这时当defer语句被执行时，跟在defer后面的函数会被延迟执行。直到包含该defer语句的函数执行完毕时，defer后的函数才会被执行。可以在一个函数中执行多条defer语句，他们执行顺序与声明顺序相反，类似于栈。

defer语句经常被用于处理成对的操作，如打开、关闭、连接、断开连接、加锁、释放锁。通过defer机制，不论函数逻辑多复杂，都能保证在任何执行路径下，资源被释放。释放资源的defer应该直接跟在请求资源的语句后。

defer后的函数会在return或者异常后执行，可以用来记录函数返回值。同样可以避免函数出现异常返回，导致文件没有关闭，或者简化代码。

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
src, err := os.Open(srcName)
if err != nil {
return
}
defer src.Close()

dst, err := os.Create(dstName)
if err != nil {
return
}
defer dst.Close()

return io.Copy(dst, src)
}

```

