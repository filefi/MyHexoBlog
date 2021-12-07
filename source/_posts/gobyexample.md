---
title: Go by Example
date: 2021-12-07 11:11:57
tags: Golang
---

# Hello World
我们的第一个程序将打印传说中的 "hello world" 消息，右边是完整的程序代码。
```go
package main

import "fmt"

func main() {
    fmt.Println("hello world")
}
```

要运行这个程序，将这些代码放到 `hello-world.go` 中并且使用 `go run` 命令。

```bash
$ go run hello-world.go
hello world
```

有时候我们想将我们的程序编译成二进制文件。我们可以通过 go build 命来达到目的。
```bash
$ go build hello-world.go
$ ls
hello-world	hello-world.go
```

然后我们可以直接运行这个二进制文件。
```bash
$ ./hello-world
hello world
```

# 值

```go
package main

import "fmt"

func main() {
	// 字符串可以通过 + 连接。
    fmt.Println("go" + "lang")
	
	// 整数和浮点数
    fmt.Println("1+1 =", 1+1)
    fmt.Println("7.0/3.0 =", 7.0/3.0)

	// 布尔型，还有你想要的逻辑运算符。
    fmt.Println(true && false)
    fmt.Println(true || false)
    fmt.Println(!true)
}
```

```bash
$ go run values.go
golang
1+1 = 2
7.0/3.0 = 2.3333333333333335
false
true
false
```

# 变量

```go
package main

import "fmt"

func main() {
	// var 声明 1 个或者多个变量。
    var a string = "initial"
    fmt.Println(a)
    
	// 你可以申明一次性声明多个变量。
    var b, c int = 1, 2
    fmt.Println(b, c)
    
	// Go 将自动推断已经初始化的变量类型。
    var d = true
    fmt.Println(d)
	
    // 声明变量且没有给出对应的初始值时，变量将会初始化为零值 。例如，一个 int 的零值是 0。
    var e int
    fmt.Println(e)

    // := 语句是申明并初始化变量的简写，例如这个例子中的 var f string = "short"。
    f := "short"
    fmt.Println(f)
}
```

```bash
$ go run variables.go
initial
1 2
true
0
short
```



# 变量

```go
package main

import "fmt"
import "math"

// const 用于声明一个常量。
const s string = "constant"

func main() {
    fmt.Println(s)
    
	// const 语句可以出现在任何 var 语句可以出现的地方
    const n = 500000000

    //常数表达式可以执行任意精度的运算
    const d = 3e20 / n
    fmt.Println(d)

    //数值型常量是没有确定的类型的，直到它们被给定了一个类型，比如说一次显示的类型转化。
    fmt.Println(int64(d))

    //当上下文需要时，一个数可以被给定一个类型，比如变量赋值或者函数调用。举个例子，这里的 math.Sin函数需要一个 float64 的参数。
    fmt.Println(math.Sin(n))
}
```

```bash
$ go run constant.go 
constant
6e+11
600000000000
-0.28470407323754404
```

# for循环

`for` 是 Go 中唯一的循环结构。这里有 `for` 循环的三个基本使用方式。

```go
package main

import "fmt"

func main() {
	// 最常用的方式，带单个循环条件。
    i := 1
    for i <= 3 {
        fmt.Println(i)
        i = i + 1
    }
    
	// 经典的初始化/条件/后续形式 for 循环。
    for j := 7; j <= 9; j++ {
        fmt.Println(j)
    }

    // 不带条件的 for 循环将一直执行，直到在循环体内使用了 break 或者 return 来跳出循环。
    for {
        fmt.Println("loop")
        break
    }
}
```

```bash
$ go run for.go
1
2
3
7
8
9
loop
```

# if/else 分支

注意，在 Go 中，你可以不适用圆括号，但是花括号是需要的。Go 里没有[三目运算符](http://en.wikipedia.org/wiki/%3F:)，所以即使你只需要基本的条件判断，你仍需要使用完整的 `if` 语句。

```go
package main

import "fmt"

func main() {
    if 7%2 == 0 {
        fmt.Println("7 is even")
    } else {
        fmt.Println("7 is odd")
    }

    // 你可以不要 else 只用 if 语句。
    if 8%4 == 0 {
        fmt.Println("8 is divisible by 4")
    }
    
	// 在条件语句之前可以有一个语句；任何在这里声明的变量都可以在所有的条件分支中使用。
    if num := 9; num < 0 {
        fmt.Println(num, "is negative")
    } else if num < 10 {
        fmt.Println(num, "has 1 digit")
    } else {
        fmt.Println(num, "has multiple digits")
    }
}
```

```bash
$ go run if-else.go 
7 is odd
8 is divisible by 4
9 has 1 digit
```

# switch 语句

```go
package main

import "fmt"
import "time"

func main() {
	// 一个基本的 switch。
    i := 2
    fmt.Print("write ", i, " as ")
    switch i {
    case 1:
        fmt.Println("one")
    case 2:
        fmt.Println("two")
    case 3:
        fmt.Println("three")
    }

	// 在一个 case 语句中，你可以使用逗号来分隔多个表达式。在这个例子中，我们很好的使用了可选的default 分支。
    switch time.Now().Weekday() {
    case time.Saturday, time.Sunday:
        fmt.Println("it's the weekend")
    default:
        fmt.Println("it's a weekday")
    }

	// 不带表达式的 switch 是实现 if/else 逻辑的另一种方式。这里展示了 case 表达式是如何使用非常量的。
    t := time.Now()
    switch {
    case t.Hour() < 12:
        fmt.Println("it's before noon")
    default:
        fmt.Println("it's after noon")
    }
}
```

```bash
$ go run switch.go 
write 2 as two
it's the weekend
it's before noon
```

# 数组

在 Go 中，**数组** 是一个固定长度的数列。
```go
package main

import "fmt"

func main() {
	// 这里我们创建了一个数组 a 来存放刚好 5 个 int。元素的类型和长度都是数组类型的一部分。数组默认是零值的，对于 int 数组来说也就是 0。
    var a [5]int
    fmt.Println("emp:", a)
    
	// 我们可以使用 array[index] = value 语法来设置数组指定位置的值，或者用 array[index] 得到值。
    a[4] = 100
    fmt.Println("set:", a)
    fmt.Println("get:", a[4])
    
	// 使用内置函数 len 返回数组的长度
    fmt.Println("len:", len(a))
    
	// 使用这个语法在一行内初始化一个数组
    var b [5]int = [5]int {1, 2, 3, 4, 5}
    // 上面语句可简写为 var b = [5]int {1, 2, 3, 4, 5}
    // 进一步简写为 b := [5]int {1, 2, 3, 4, 5}
    
    fmt.Println("dcl:", b)
    
	// 数组的存储类型是单一的，但是你可以组合这些数据来构造多维的数据结构。
    var twoD [2][3]int
    for i := 0; i < 2; i++ {
        for j := 0; j < 3; j++ {
            twoD[i][j] = i + j
        }
    }
    fmt.Println("2d: ", twoD) // 注意，在使用 fmt.Println 来打印数组的时候，会使用[v1 v2 v3 ...] 的格式显示
}
```

```bash
$ go run arrays.go
emp: [0 0 0 0 0]
set: [0 0 0 0 100]
get: 100
len: 5
dcl: [1 2 3 4 5]
2d:  [[0 1 2] [1 2 3]]
```
在典型的 Go 程序中，相对于数组而言，**slice** 使用的更多。我们将在后面讨论 **slice**。



# 切片
**Slice** 是 Go 中一个关键的数据类型，是一个比数组更加强大的序列接口。

```go
package main

import "fmt"

func main() {
	// 不像数组，slice 的类型仅由它所包含的元素决定（不像数组中还需要元素的个数）。
    // 要创建一个长度非零的空slice，需要使用内建的方法 make。
    // 这里我们创建了一个长度为3的 string 类型 slice（初始化为零值）。
    s := make([]string, 3)
    fmt.Println("emp:", s) // emp: [  ]
	
    // 我们可以和数组一样设置和得到值
    s[0] = "a"
    s[1] = "b"
    s[2] = "c"
    fmt.Println("set:", s) // set: [a b c]
    fmt.Println("get:", s[2]) // get: c

    // 如你所料，len 返回 slice 的长度
    fmt.Println("len:", len(s)) // len: 3
    
	// 作为基本操作的补充，slice 支持比数组更多的操作。其中一个是内建的 append，它返回一个包含了一个或者多个新值的 slice。注意我们接受返回由 append返回的新的 slice 值。
    s = append(s, "d")
    s = append(s, "e", "f")
    fmt.Println("apd:", s) // apd: [a b c d e f]

    // Slice 也可以被 copy。这里我们创建一个空的和 s 有相同长度的 slice c，并且将 s 复制给 c。
    c := make([]string, len(s))
    copy(c, s)
    fmt.Println("cpy:", c) // cpy: [a b c d e f]
	
    // Slice 支持通过 slice[low:high] 语法进行“切片”操作。例如，这里得到一个包含元素 s[2], s[3],s[4] 的 slice。
    l := s[2:5]
    fmt.Println("sl1:", l) // sl1: [c d e]

    // 这个 slice 从 s[0] 到（但是包含）s[5]。
    l = s[:5]
    fmt.Println("sl2:", l) // sl2: [a b c d e]

    // 这个 slice 从（包含）s[2] 到 slice 的后一个值。
    l = s[2:]
    fmt.Println("sl3:", l) // sl3: [c d e f]

    // 我们可以在一行代码中声明并初始化一个 slice 变量。
    var t []string = []string {"g", "h", "i"}
	// 上面语句可简写为 var t = []string {"g", "h", "i"}
	// 进一步简写为 t := []string{"g", "h", "i"}
    fmt.Println("dcl:", t) // dcl: [g h i]

    // Slice 可以组成多维数据结构。内部的 slice 长度可以不同，这和多位数组不同。
    twoD := make([][]int, 3)
    for i := 0; i < 3; i++ {
        innerLen := i + 1
        twoD[i] = make([]int, innerLen)
        for j := 0; j < innerLen; j++ {
            twoD[i][j] = i + j
        }
    }
    fmt.Println("2d: ", twoD) // 2d:  [[0] [1 2] [2 3 4]]
    // 注意，slice 和数组不同，虽然它们通过 fmt.Println 输出差不多。
}
```

```bash
$ go run slices.go
emp: [  ]
set: [a b c]
get: c
len: 3
apd: [a b c d e f]
cpy: [a b c d e f]
sl1: [c d e]
sl2: [a b c d e]
sl3: [c d e f]
dcl: [g h i]
2d:  [[0] [1 2] [2 3 4]]
```
看看这个由 Go 团队撰写的一篇[很棒的博文](http://blog.golang.org/2011/01/go-slices-usage-and-internals.html)，获得更多关于 Go 中 slice 的设计和实现细节。

# map

现在，我们已经看过了数组和 slice，接下来我们将看看 Go 中的另一个关键的内建数据类型：map。

map 是 Go 内置关联数据类型（在一些其他的语言中称为哈希 或者字典 ）。

```go
package main

import "fmt"

func main() {
	// 要创建一个空 map，需要使用内建的 make:make(map[key-type]val-type).
    m := make(map[string]int)
    
    // 使用典型的 make[key] = val 语法来设置键值对。
    m["k1"] = 7
    m["k2"] = 13
    
	// 使用例如 Println 来打印一个 map 将会输出所有的键值对。
    fmt.Println("map:", m) // map: map[k1:7 k2:13]
    // 注意一个 map 在使用 fmt.Println 打印的时候，是以 map[k:v k:v]的格式输出的。
    
	// 使用 name[key] 来获取一个键的值
    v1 := m["k1"]
    fmt.Println("v1: ", v1) // v1:  7
    
	// 当对一个 map 调用内建的 len 时，返回的是键值对数目
    fmt.Println("len:", len(m)) // len: 2
    
	// 内建的 delete 可以从一个 map 中移除键值对
    delete(m, "k2")
    fmt.Println("map:", m) // map: map[k1:7]
    
	// 当从一个 map 中取值时，可选的第二返回值指示这个键是在这个 map 中。这可以用来消除键不存在和键有零值，像 0 或者 "" 而产生的歧义。
    // _, prs := m["k2"]
    k2, prs := m["k2"]
    fmt.Println("prs:", prs) // prs: false
    fmt.Println("k2:", k2) // k2: 0

    // 你也可以通过这个语法在同一行申明和初始化一个新的map。
    n := map[string]int{"foo": 1, "bar": 2}
    fmt.Println("map:", n) // map: map[bar:2 foo:1]
}
```

```bash
$ go run maps.go 
map: map[k1:7 k2:13]
v1:  7
len: 2
map: map[k1:7]
prs: false
map: map[foo:1 bar:2]
```

# range遍历

`range` 迭代各种各样的数据结构。让我们来看看如何在我们已经学过的数据结构上使用 `range` 吧。

```go
package main

import "fmt"

func main() {
	// 这里我们使用 range 来统计一个 slice 的元素个数。数组也可以采用这种方法。
    nums := []int{2, 3, 4}
    sum := 0
    for _, num := range nums { // 这里我们不需要索引，所以使用 空值定义符_ 来忽略它。有时候我们实际上是需要这个索引的。
        sum += num
    }
    fmt.Println("sum:", sum) // sum: 9
    
	// range 在数组和 slice 中都同样提供每个项的索引和值。
    for i, num := range nums {
        if num == 3 {
            fmt.Println("index:", i) // index: 1
        }
    }
    
	// range 在 map 中迭代键值对。
    kvs := map[string]string{"a": "apple", "b": "banana"}
    for k, v := range kvs {
        fmt.Printf("%s -> %s\n", k, v) 
        // a -> apple
        // b -> banana
    }
    
	// range 在字符串中迭代 unicode 编码。第一个返回值是rune 的起始字节位置，然后第二个是 rune 自己。
    for i, c := range "go" {
        fmt.Println(i, c)
        // 0 103
        // 1 111
    }
}
```

```bash
	
$ go run range.go 
sum: 9
index: 1
a -> apple
b -> banana
0 103
1 111
```

# 函数

```go
package main

import "fmt"

// 这里是一个函数，接受两个 int 并且以 int 返回它们的和
func plus(a int, b int) int {
	// Go 需要明确的返回值，例如，它不会自动返回最后一个表达式的值
    return a + b
}

func main() {
	// 正如你期望的那样，通过 name(args) 来调用一个函数，
    res := plus(1, 2)
    fmt.Println("1+2 =", res)
}
```

``` bash
$ go run functions.go 
1+2 = 3
```

## 返回多个值

Go 内建*多返回值* 支持。这个特性在 Go 语言中经常被用到，例如用来同时返回一个函数的结果和错误信息。

```go
package main

import "fmt"

// (int, int) 在这个函数中标志着这个函数返回 2 个 int。
func vals() (int, int) {
    return 3, 7
}

func main() {
	// 这里我们通过多赋值 操作来使用这两个不同的返回值。
    a, b := vals()
    fmt.Println(a)
    fmt.Println(b)

    // 如果你仅仅想返回值的一部分的话，你可以使用空白定义符 _。
    _, c := vals()
    fmt.Println(c)
}
```

```bash
$ go run multiple-return-values.go
3
7
7
```

## 可变参数函数

[*可变参数函数*](http://zh.wikipedia.org/wiki/可變參數函數)。可以用任意数量的参数调用。例如，`fmt.Println` 是一个常见的变参函数。

```go
package main

import "fmt"

// 这个函数使用任意数目的 int 作为参数。
func sum(nums ...int) {
    fmt.Print(nums, " ")
    total := 0
    for _, num := range nums {
        total += num
    }
    fmt.Println(total)
}

func main() {
	// 变参函数使用常规的调用方式，除了参数比较特殊。
    sum(1, 2)
    sum(1, 2, 3)
    
	// 如果你的 slice 已经有了多个值，想把它们作为变参使用，你要这样调用 func(slice...)。
    nums := []int{1, 2, 3, 4}
    sum(nums...)
}
```

```bash
$ go run variadic-functions.go 
[1 2] 3
[1 2 3] 6
[1 2 3 4] 10
```

## 闭包

Go 支持通过 [*闭包*](http://zh.wikipedia.org/wiki/闭包_(计算机科学)) 来使用 [*匿名函数*](http://zh.wikipedia.org/wiki/匿名函数)。匿名函数在你想定义一个不需要命名的内联函数时是很实用的。

```go
package main

import "fmt"

// 这个 intSeq 函数返回另一个在 intSeq 函数体内定义的匿名函数。
// 这个返回的匿名函数返回一个int类型；
// 同时，这个返回的函数使用闭包的方式 隐藏 变量 i。
func intSeq() func() int {
    i := 0
    return func() int {
        i += 1
        return i
    }
}

func main() {
	//我们调用 intSeq 函数，将返回值（也是一个函数）赋给nextInt。
    // 这个函数的值包含了自己的值 i，这样在每次调用 nextInt 时都会更新 i 的值。
    nextInt := intSeq()
    
	// 通过多次调用 nextInt 来看看闭包的效果。
    fmt.Println(nextInt()) // 1
    fmt.Println(nextInt()) // 2
    fmt.Println(nextInt()) // 3

    // 为了确认这个状态对于这个特定的函数是唯一的，我们重新创建并测试一下。
    newInts := intSeq()
    fmt.Println(newInts()) // 1
}
```

```bash
$ go run closures.go
1
2
3
1
```

## 递归

```go
package main

import "fmt"

// 这里是一个经典的阶乘示例。
func fact(n int) int {
    if n == 0 { // face 函数在到达 face(0) 前一直调用自身。
        return 1
    }
    return n * fact(n-1)
}

func main() {
    fmt.Println(fact(7))
}
```

```bash
$ go run recursion.go 
5040
```

# 指针

Go 支持 *[指针](http://zh.wikipedia.org/wiki/指標_(電腦科學))*，允许在程序中通过引用传递值或者数据结构。

```go
package main

import "fmt"

// 我们将通过两个函数：zeroval 和 zeroptr 来比较指针和值类型的不同。

// zeroval 有一个 int 型参数，所以使用值传递。zeroval 将从调用它的那个函数中得到一个 ival形参的拷贝。
func zeroval(ival int) {
    ival = 0
}

// zeroptr 有一个和上面不同的 *int 参数，意味着它用了一个 int指针。
// 函数体内的 *iptr 接着解引用 这个指针，从它内存地址得到这个地址对应的当前值。
// 对一个解引用的指针赋值将会改变这个指针引用的真实地址的值。
func zeroptr(iptr *int) {
    *iptr = 0
}

func main() {
    i := 1
    fmt.Println("initial:", i)
    zeroval(i)
    fmt.Println("zeroval:", i)
    
	// 通过 &i 语法来取得 i 的内存地址，例如一个变量i 的指针。
    zeroptr(&i)
    fmt.Println("zeroptr:", i)

    // 指针也是可以被打印的。
    fmt.Println("pointer:", &i)
}
// zeroval 在 main 函数中不能改变 i 的值，但是zeroptr 可以，因为它有一个这个变量的内存地址的引用。
```

```bash
$ go run pointers.go
initial: 1
zeroval: 1
zeroptr: 0
pointer: 0x42131100
```











































































































