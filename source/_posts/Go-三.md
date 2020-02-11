title: Go(三)
author: Zhiyuan
tags:
  - Go
categories:
  - 学习笔记
date: 2019-10-03 10:16:00

---

### 复合数据类型

#### 3.1 数组

```go
var a [3]int
var q [3]int = [3]int{1, 2, 3}
q := [...]int{1, 2, 3}  //长度由初始化值的个数来计算
months := [...]string{1: "January", /* ... */, 12: "December"}  //索引0自动初始化为空字符串
```

数组类型可以直接比较（==），只有当两个数组所有元素都是相等的时候数组才是相等的。

#### 3.2 Slice

一个slice由三个部分构成：指针、长度和容量。指针指向第一个slice元素对应的底层数组元素的地址，要注意的是slice的第一个元素并不一定就是数组的第一个元素。长度对应slice中元素的数目；长度不能超过容量，容量一般是从slice的开始位置到底层数据的结尾位置。内置的len和cap函数分别返回slice的长度和容量。

slice的切片操作s[i:j]，其中0 ≤ i≤ j≤ cap(s)，用于创建一个新的slice，引用s的从第i个元素开始到第j-1个元素的子序列。

slice之间不能比较，因此我们不能使用==操作符来判断两个slice是否含有全部相等元素。不过标准库提供了高度优化的bytes.Equal函数来判断两个字节型slice是否相等（[]byte），但是对于其他类型的slice，我们必须自己展开每个元素进行比较：

```go
func equal(x, y []string) bool {
    if len(x) != len(y) {
        return false
    }
    for i := range x {
        if x[i] != y[i] {
            return false
        }
    }
    return true
}
```

slice唯一合法的比较操作是和nil比较，内置的make函数创建一个指定元素类型、长度和容量的slice。

```go
var s []int    // len(s) == 0, s == nil
s = nil        // len(s) == 0, s == nil
s = []int(nil) // len(s) == 0, s == nil
s = []int{}    // len(s) == 0, s != nil
make([]T, len)   //返回整个数组
make([]T, len, cap) // same as make([]T, cap)[:len]
```

#### 3.3 append函数

内置的append函数用于向slice追加元素

```go
var runes []rune
for _, r := range "Hello, 世界" {
    runes = append(runes, r)
}
fmt.Printf("%q\n", runes) // "['H' 'e' 'l' 'l' 'o' ',' ' ' '世' '界']"
```

copy函数可以方便地将一个slice复制另一个相同类型的slice。copy函数的第一个参数是要复制的目标slice，第二个参数是源slice。`copy(z, x)`

```go
var x []int
x = append(x, 1)
x = append(x, 2, 3)
x = append(x, 4, 5, 6)
x = append(x, x...) // append the slice x
```

一个slice可以用来模拟一个stack。最初给定的空slice对应一个空的stack，然后可以使用append函数将新的值压入stack：

```go
stack = append(stack, v) // push v
top := stack[len(stack)-1] // top of stack
stack = stack[:len(stack)-1] // pop
```

要删除slice中间的某个元素并保存原有的元素顺序，可以通过内置的copy函数将后面的子slice向前依次移动一位完成：

```go
func remove(slice []int, i int) []int {
    copy(slice[i:], slice[i+1:])
    return slice[:len(slice)-1]
}

func main() {
    s := []int{5, 6, 7, 8, 9}
    fmt.Println(remove(s, 2)) // "[5 6 8 9]"
}
```

#### 3.4 Map

在Go语言中，一个map就是一个哈希表的引用，map类型可以写为map[K]V，其中K和V分别对应key和value。map中所有的key都有相同的类型，所有的value也有着相同的类型，但是key和value之间可以是不同的数据类型。其中K对应的key必须是支持==比较运算符的数据类型，所以map可以通过测试key是否相等来判断是否已经存在。

```go
ages := make(map[string]int) // mapping from strings to ints
ages := map[string]int{
    "alice":   31,
    "charlie": 34,
}
//创建空的map
map[string]int{}
//使用内置的delete函数可以删除元素
delete(ages, "alice") // remove element ages["alice"]
```

map中的元素并不是一个变量，因此我们不能对map的元素进行取址操作，原因是map可能随着元素数量的增长而重新分配更大的内存空间，从而可能导致之前的地址无效。

```go
_ = &ages["bob"] // compile error: cannot take address of map element

```

**遍历map**

```go
for name, age := range ages {
    fmt.Printf("%s\t%d\n", name, age)
}

```

Map的迭代顺序是不确定的，并且不同的哈希函数实现可能导致不同的遍历顺序。在实践中，遍历的顺序是随机的，每一次遍历的顺序都不相同。如果要按顺序遍历key/value对，我们必须显式地对key进行排序，可以使用sort包的Strings函数对字符串slice进行排序。

```go
import "sort"

var names []string
for name := range ages {
    names = append(names, name)
}
sort.Strings(names)
for _, name := range names {
    fmt.Printf("%s\t%d\n", name, ages[name])
}

```

```go
age, ok := ages["bob"]
//在这种场景下，map的下标语法将产生两个值；第二个是一个布尔值，
//用于报告元素是否真的存在。布尔变量一般命名为ok，特别适合马上用于if条件判断部分。

```

Go语言没有提供set类型，可以用map实现set类型，

```go
func main() {
    seen := make(map[string]bool) // a set of strings
    input := bufio.NewScanner(os.Stdin)
    for input.Scan() {
        line := input.Text()
        if !seen[line] {
            seen[line] = true
            fmt.Println(line)
        }
    }

    if err := input.Err(); err != nil {
        fmt.Fprintf(os.Stderr, "dedup: %v\n", err)
        os.Exit(1)
    }
}

```

#### 3.5 结构体

```Go
type Employee struct {
    ID        int
    Name      string
    Address   string
    DoB       time.Time
}

var dilbert Employee

```

结构体变量的成员可以通过点操作符访问，比如dilbert.Name和dilbert.DoB。赋值可以直接进行或者通过指针：

```Go
name := &dilbert.Name
*name = "Robin"

```

点操作符也可以和指向结构体的指针一起工作：

```Go
var employeeOfTheMonth *Employee = &dilbert
employeeOfTheMonth.Position += " (proactive team player)"
//上面语句等价于
(*employeeOfTheMonth).Position += " (proactive team player)"

```

结构体成员的输入顺序也有重要的意义。交换Name和Address出现的先后顺序，那样的话就是定义了不同的结构体类型。如果结构体成员名字是以大写字母开头的，那么该成员就是导出的；这是Go语言导出规则决定的。一个命名为S的结构体类型将不能再包含S类型的成员，但是S类型的结构体可以包含`*S`指针类型的成员，这可以让我们创建递归的数据结构，比如链表和树结构等。

##### 3.5.1 结构体字面值

```Go
type Point struct{ X, Y int }

p := Point{1, 2}
q := Point{X:1,Y:2}	

```

上面第一种写法，要求以结构体成员定义的顺序为每个结构体成员指定一个字面值。这个方法一般只在定义结构体的包内部使用，或者是在较小的结构体中使用。第二种方法是以成员名字和相应的值来初始化。结构体通常通过指针处理，可以用下面的写法来创建并初始化一个结构体变量，并返回结构体的地址：

```Go
pp := &Point{1, 2}
//等价于
pp := new(Point)
*pp = Point{1, 2}

```

##### 3.5.2 结构体比较

如果结构体的全部成员都是可以比较的，那么这个结构体也是可以比较的，可以通过==或!=运算符进行比较。

```Go
type Point struct{ X, Y int }

p := Point{1, 2}
q := Point{2, 1}
fmt.Println(p.X == q.X && p.Y == q.Y) // "false"
fmt.Println(p == q)                   // "false"

```

可比较的结构体类型和其他可比较的类型一样，可以用于map的key类型。

```Go
type address struct {
    hostname string
    port     int
}

hits := make(map[address]int)
hits[address{"golang.org", 443}]++

```

##### 3.5.3 结构体嵌入和匿名变量

```Go
type Point struct {
    X, Y int
}

type Circle struct {
    Center Point
    Radius int
}

type Wheel struct {
    Circle Circle
    Spokes int
}
var w Wheel
w.Circle.Center.X = 8
w.Circle.Center.Y = 8
w.Circle.Radius = 5
w.Spokes = 20

```

Go语言有一个特性让我们只声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员。匿名成员的数据类型必须是命名的类型或指向一个命名的类型的指针。得意于匿名嵌入的特性，我们可以直接访问叶子属性而不需要给出完整的路径：

```Go
type Circle struct {
    Point
    Radius int
}

type Wheel struct {
    Circle
    Spokes int
}
var w Wheel
w.X = 8         // equivalent to w.Circle.Point.X = 8
w.Y = 8         // equivalent to w.Circle.Point.Y = 8
w.Radius = 5    // equivalent to w.Circle.Radius = 5
w.Spokes = 20
//匿名成员Circle和Point都有自己的名字——就是命名的类型名字——但是这些名字在点操作符中是可选的。

```

结构体字面值没有简短表示匿名成员的语法，所以必须遵循形状类型声明时的结构，按照以下方式赋值：

```Go
w = Wheel{Circle{Point{8, 8}, 5}, 20}

w = Wheel{
    Circle: Circle{
        Point:  Point{X: 8, Y: 8},
        Radius: 5,
    },
    Spokes: 20, // NOTE: trailing comma necessary here (and at Radius)
}

```

因为匿名成员也有一个隐式的名字，因此不能同时包含两个类型相同的匿名成员，这会导致名字冲突。同时，因为成员的名字是由其类型隐式地决定的，所有匿名成员也有可见性的规则约束。在上面的例子中，Point和Circle匿名成员都是导出的。即使它们不导出（比如改成小写字母开头的point和circle），我们依然可以用简短形式访问匿名成员嵌套的成员，但是在包外部，这是不允许的。

```Go
w.X = 8 // equivalent to w.circle.point.X = 8

```

#### 3.6 JSON(encoding/json包)

```Go
type Movie struct {
    Title  string
    Year   int  `json:"released"`
    Color  bool `json:"color,omitempty"`
    Actors []string
}

var movies = []Movie{
    {Title: "Casablanca", Year: 1942, Color: false,
        Actors: []string{"Humphrey Bogart", "Ingrid Bergman"}},
    {Title: "Cool Hand Luke", Year: 1967, Color: true,
        Actors: []string{"Paul Newman"}},
    {Title: "Bullitt", Year: 1968, Color: true,
        Actors: []string{"Steve McQueen", "Jacqueline Bisset"}},
    // ...
}

```

结构体声明中，Year和Color成员后面的字符串面值是结构体成员的Tag。将一个Go语言中类似movies的结构体slice转为JSON的过程叫编组（marshaling）。编组通过调用json.Marshal函数完成：

```Go
data, err := json.Marshal(movies)
if err != nil {
    log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)

```

Marshal函数返还一个编码后的字节slice，包含很长的字符串，并且没有空白缩进。这种紧凑的表示形式虽然包含了全部的信息，但是很难阅读。为了生成便于阅读的格式，另一个json.MarshalIndent函数将产生整齐缩进的输出。该函数有两个额外的字符串参数用于表示每一行输出的前缀和每一个层级的缩进。

```Go
data, err := json.MarshalIndent(movies, "", "    ")
if err != nil {
    log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)
//输出格式
[
    {
        "Title": "Casablanca",
        "released": 1942,
        "Actors": [
            "Humphrey Bogart",
            "Ingrid Bergman"
        ]
    },
    ………………

```

其中Year名字的成员在编码后变成了released，还有Color成员编码后变成了小写字母开头的color。这是因为构体成员Tag所导致的。结构体的成员Tag可以是任意的字符串面值，但是通常是一系列用空格分隔的key:"value"键值对序列；因为值中含义双引号字符，因此成员Tag一般用原生字符串面值的形式书写。**json开头键名**对应的值用于控制encoding/json包的编码和解码的行为，并且encoding/...下面其它的包也遵循这个约定。成员Tag中json对应值的第一部分用于指定JSON对象的名字，Color成员的Tag还带了一个额外的omitempty选项，表示当Go语言结构体成员为空或零值时不生成JSON对象，如Casablanca的color成员变量值为false（零值），所以没有输出color成员。

解码操作是将JSON数据解码成Go语言的数据结构，通过**json.Unmarshal**函数完成。下面的代码将JSON格式的电影数据解码为一个结构体slice，结构体中只有Title成员。通过定义合适的Go语言数据结构，我们可以**选择性地解码**JSON中感兴趣的成员。当Unmarshal函数调用返回，slice将被只含有Title信息值填充，其它JSON成员将被忽略。

```Go
var titles []struct{ Title string }
if err := json.Unmarshal(data, &titles); err != nil {
    log.Fatalf("JSON unmarshaling failed: %s", err)
}
fmt.Println(titles) // "[{Casablanca} {Cool Hand Luke} {Bullitt}]"

```

#### 3.7 文本和HTML模板

text\\template和html\\template提供了一个将变量值填充到文本或HTML格式的模板的机制。

一个模板是一个字符串或一个文件，里面包含了一个或多个由双花括号包含的`{{action}}`对象。大部分的字符串只是按字面值打印，但是对于actions部分将触发其它的行为。下面是一个简短的模板字符串。

```Go
const templ = `{{.TotalCount}} issues:
{{range .Items}}----------------------------------------
Number: {{.Number}}
User:   {{.User.Login}}
Title:  {{.Title | printf "%.64s"}}
Age:    {{.CreatedAt | daysAgo}} days
{{end}}`

```

```
对于每一个action，都有一个当前值的概念，对应点操作符，写作“.”。模板中{{.TotalCount}}对应action将展开为结构体中TotalCount成员以默认的方式打印的值。模板中{{range .Items}}和{{end}}对应一个循环action，因此它们直接的内容可能会被展开多次，循环每次迭代的当前值对应当前的Items元素的值。
```

在一个action中，`|`操作符表示将前一个表达式的结果作为后一个函数的输入，在Title这一行的action中，第二个操作是一个printf函数，是一个基于fmt.Sprintf实现的内置函数，所有模板都可以直接使用。

生成模板：template.New先创建并返回一个模板；Funcs方法将daysAgo等自定义函数注册到模板中，并返回模板；最后调用Parse函数分析模板。

```Go
report, err := template.New("report").
    Funcs(template.FuncMap{"daysAgo": daysAgo}).
    Parse(templ)
if err != nil {
    log.Fatal(err)
}

```

https://books.studygolang.com/gopl-zh/ch4/ch4-06.html