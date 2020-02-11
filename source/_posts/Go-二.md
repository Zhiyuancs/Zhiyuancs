---
title: Go(二)
date: 2019-09-26 15:44:51
tags:
	Go
categories:
	学习笔记
---

## 基础数据类型

#### 2.1 整形

```go
*      /      %      <<       >>     &       &^
+      -      |      ^
==     !=     <      <=       >      >=
&&
||
```

二元运算符有五种优先级。在同一个优先级，使用左优先结合规则，但是使用括号可以明确优先顺序，使用括号也可以用于提升优先级。在Go语言中，%取模运算符的符号和被取模数的符号总是一致的，因此`-5%3`和`-5%-3`结果都是-2。除法运算符`/`的行为则依赖于操作数是否为全为整数，比如`5.0/4.0`的结果是1.25，但是5/4的结果是1，因为整数除法会向着0方向截断余数。

```go
&      位运算 AND
|      位运算 OR
^      位运算 XOR
&^     位清空 (AND NOT)
<<     左移
>>     右移
```

位操作运算符`^`作为二元运算符时是按位异或（XOR），当用作一元运算符时表示按位取反；位操作运算符`&^`用于按位置零（AND NOT）：如果对应y中bit位为1的话, 表达式`z = x &^ y`结果z的对应的bit位为0，否则z对应的bit位等于x相应的bit位的值。

任何大小的整数字面值都可以用以0开始的八进制格式书写，例如0666；或用以0x或0X开头的十六进制格式书写，例如0xdeadbeef。十六进制数字可以用大写或小写字母。

```go
o := 0666
fmt.Printf("%d %[1]o %#[1]o\n", o) // "438 666 0666"
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x)
// Output:
// 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

通常Printf格式化字符串包含多个%参数时将会包含对应相同数量的额外操作数，但是%之后的`[1]`副词告诉Printf函数再次使用第一个操作数。第二，%后的`#`副词告诉Printf在用%o、%x或%X输出时生成0、0x或0X前缀。字符使用`%c`参数打印，或者是用`%q`参数打印带单引号的字符。

#### 2.2 浮点数

float32、float64

很小或很大的数最好用科学计数法书写，通过e或E来指定指数部分。

```go
const Avogadro = 6.02214129e23  // 阿伏伽德罗常数
```

用Printf函数的%g参数打印浮点数，将采用更紧凑的表示形式打印，并提供足够的精度，但是对应表格的数据，使用%e（带指数）或%f的形式打印可能更合适。所有的这三个打印形式都可以指定打印的宽度和控制打印精度。

```go
for x := 0; x < 8; x++ {
    fmt.Printf("x = %d e^x = %8.3f\n", x, math.Exp(float64(x)))
}

x = 5       e^x =  148.413
x = 6       e^x =  403.429
x = 7       e^x = 1096.633
```

math包中有定义的特殊值：+Inf、-Inf、NAN（正负无穷，非数）。函数math.IsNaN用于测试一个数是否是非数NaN，math.NaN则返回非数对应的值。

```go
nan := math.NaN()
fmt.Println(nan == nan, nan < nan, nan > nan) // "false false false"
```

#### 2.3 复数

Go语言提供了两种精度的复数类型：complex64和complex128，分别对应float32和float64两种浮点数精度。内置的complex函数用于构建复数，内建的**real**和**imag**函数分别返回复数的实部和虚部：

```go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y)                 // "(-5+10i)"
fmt.Println(real(x*y))           // "-5"
fmt.Println(imag(x*y))           // "10"
```

math/cmplx包提供了复数处理的许多函数，例如求复数的平方根函数和求幂函数。

#### 2.4 布尔值

布尔值可以和&&（AND）和||（OR）操作符结合，并且有短路行为：如果运算符左边值已经可以确定整个布尔表达式的值，那么运算符右边的值将不再被求值。

`&&`的优先级比`||`高

布尔值并不会隐式转换为数字值0或1，反之亦然。

#### 2.5 字符串

支持切片，‘+’链接字符串，字符串是不可修改的，尝试修改字符串内部的字符是禁止的。

原生字符串使用反引号代替双引号，原生的字符串字面量多用于书写多行消息、HTML以及正则表达式。在原生的字符串面值中，没有转义操作；**全部的内容都是字面的意思**，包含退格和换行，因此一个程序中的原生字符串面值可能跨越多行。

```go
const GoUsage = `Go is a tool for managing Go source code.

Usage:
    go command [arguments]
...`
```

strings包提供了许多如字符串的查询、替换、比较、截断、拆分和合并等功能。bytes包也提供了很多类似功能的函数，但是针对和字符串有着相同结构的[]byte类型。

```go
//strings包
func Contains(s, substr string) bool
func Count(s, sep string) int
func Fields(s string) []string
func HasPrefix(s, prefix string) bool
func Index(s, sep string) int
func Join(a []string, sep string) string
//bytes包
func Contains(b, subslice []byte) bool
func Count(s, sep []byte) int
func Fields(s []byte) [][]byte
func HasPrefix(s, prefix []byte) bool
func Index(s, sep []byte) int
func Join(s [][]byte, sep []byte) []byte

```

bytes包还提供了**Buffer类型**用于字节slice的缓存。一个Buffer开始是空的，但是随着string、byte或[]byte等类型数据的写入可以动态增长，一个bytes.Buffer变量并不需要初始化，因为零值也是有效的：

```go
var buf bytes.Buffer
buf.WriteByte('[')
buf.WriteRune('中')
buf.WriteString(", d ")

```

strconv包提供了布尔型、整型数、浮点数和对应字符串的相互转换，还提供了双引号转义相关的转换。unicode包提供了IsDigit、IsLetter、IsUpper和IsLower等类似功能，它们用于给字符分类。每个函数有一个单一的rune类型的参数，然后返回一个布尔值。

#### 2.6 UTF-8

UTF8编码使用1到4个字节来表示每个Unicode码点，ASCII部分字符只使用1个字节，常用字符部分使用2或3个字节表示。如果第一个字节的高端bit为0，则表示对应7bit的ASCII字符，ASCII字符每个字符依然是一个字节，和传统的ASCII编码兼容。如果第一个字节的高端bit是110，则说明需要2个字节；后续的每个高端bit都以10开头。

```
0xxxxxxx                             runes 0-127    (ASCII)
110xxxxx 10xxxxxx                    128-2047       (values <128 unused)
1110xxxx 10xxxxxx 10xxxxxx           2048-65535     (values <2048 unused)
11110xxx 10xxxxxx 10xxxxxx 10xxxxxx  65536-0x10ffff (other values unused)

```

```
//Unicode转义字符让我们可以通过Unicode码点输入特殊的字符。下面是相同的字符串
"世界"
"\xe4\xb8\x96\xe7\x95\x8c"
"\u4e16\u754c"
"\U00004e16\U0000754c"

```

```go
import "unicode/utf8"

s := "Hello, 世界"
fmt.Println(len(s))                    // "13"
fmt.Println(utf8.RuneCountInString(s)) // "9"

```

Go语言的range循环在处理字符串的时候，会自动隐式解码UTF8字符串。

如果是将一个[]rune类型的Unicode字符slice或数组转为string，则对它们进行UTF8编码：

```go
s := "プログラム"
fmt.Printf("% x\n", s) // "e3 83 97 e3 83 ad e3 82 b0 e3 83 a9 e3 83 a0"
r := []rune(s)
fmt.Printf("%x\n", r)  // "[30d7 30ed 30b0 30e9 30e0]"
fmt.Println(string(r)) // "プログラム"
fmt.Println(string(65))     // "A", not "65"
fmt.Println(string(0x4eac)) // "京"
fmt.Println(string(1234567)) // "�"（无效字符）
fmt.Println(rune('你'))//输出20320

```

rune在golang中是int32的别名，在各个方面都与int32相同。可以将rune理解为一个可以表示Unicode编码的int值，称为码点。参考java中的char类型（可以和int运算）。

在Go中，双引号是用来表示字符串string，本质是[]byte类型，单引号表示rune类型。

#### 2.7 字符串和数字的转换

将一个整数转为字符串，一种方法是用fmt.Sprintf返回一个格式化的字符串；另一个方法是用strconv.Itoa(“整数到ASCII”)：

```go
x := 123
y := fmt.Sprintf("%d", x)
fmt.Println(y, strconv.Itoa(x)) // "123 123"

```

FormatInt和FormatUint函数可以用不同的进制来格式化数字：

```go
fmt.Println(strconv.FormatInt(int64(x), 2)) // "1111011"

```

fmt.Sprintf函数的%b、%d、%o和%x等参数提供功能往往比strconv包的Format函数方便很多，特别是在需要包含附加额外信息的时候：

```go
s := fmt.Sprintf("x=%b", x) // "x=1111011"

```

如果要将一个字符串解析为整数，可以使用strconv包的Atoi或ParseInt函数，还有用于解析无符号整数的ParseUint函数：

```go
x, err := strconv.Atoi("123")             // x is an int
y, err := strconv.ParseInt("123", 10, 64) // base 10, up to 64 bits
func ParseInt(s string, base int, bitSize int) (i int64, err error)

```

ParseInt函数的第三个参数是用于指定返回整型数的大小；例如16表示int16，0则表示int。第二个参数为数字字符串的进制。

int随系统而定，32位系统为int32

#### 2.8 常量

批量声明

```go
const (
    e  = 2.7182818284590452
    e1
    pi = 3.1415926535897932
    p3
)
//p3=pi,e1=e


```

**iota 常量生成器**:在一个const声明语句中，在第一个声明的常量所在的行，iota将会被置为0，然后在每一个有常量声明的行加一。

```go
type Weekday int

const (
    Sunday Weekday = iota
    Monday
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
)
//周日将对应0，周一为1，如此等等

const (
    _ = 1 << (10 * iota)
    KiB // 1024
    MiB // 1048576
    GiB // 1073741824
    TiB // 1099511627776             (exceeds 1 << 32)
    PiB // 1125899906842624
    EiB // 1152921504606846976
    ZiB // 1180591620717411303424    (exceeds 1 << 64)
    YiB // 1208925819614629174706176
)

```

#### 2.9 无类型常量

有六种未明确类型的常量类型，分别是无类型的布尔型、无类型的整数、无类型的字符、无类型的浮点数、无类型的复数、无类型的字符串。

math.Pi无类型的浮点数常量，可以直接用于任意需要浮点数或复数的地方：

```go
var x float32 = math.Pi
var y float64 = math.Pi
var z complex128 = math.Pi

```

