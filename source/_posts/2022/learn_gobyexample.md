---
title: gobyexample 学习笔记
date: 2022-05-29 00:10:21
tags: Golang
categories: Golang
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

<!-- more -->

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



# 常量

```go
package main

import (
    "fmt"
    "math"
)

// const 用于声明一个常量。
const s string = "constant"

func main() {
    fmt.Println(s)

    // 重新赋值会报错
    s = "yoyo" // cannot assign to s (constant "constant" of type string)

    var s int = 100 // 重新定义局部变量则不会报错
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

	// 在一个 case 语句中，你可以使用逗号来分隔多个表达式。
  // 在这个例子中，我们很好的使用了可选的 default 分支。
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
    var b [5]int = [5]int{1, 2, 3, 4, 5}
    // 上面语句可简写为 var b = [5]int{1, 2, 3, 4, 5}
    // 进一步简写为 b := [5]int{1, 2, 3, 4, 5}
    
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
    // 要创建一个长度非零的空slice，需要使用内建函数 make。
    // 这里我们创建了一个长度为3的 string 类型 slice（初始化为零值）。
    s := make([]string, 3)
    fmt.Println("emp:", s) // emp: [  ]

    // 我们可以和数组一样设置和得到值
    s[0] = "a"
    s[1] = "b"
    s[2] = "c"
    fmt.Println("set:", s)    // set: [a b c]
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
    var t []string = []string{"g", "h", "i"}
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

```go
// array 和 slice 的字面量声明的区别在于方括号[]中是否指定长度
var a [3]string
fmt.Println(a)
append(a, "a", "b") // 此时会报错：first argument to append must be a slice; have a (variable of type [3]string)
```

# map

现在，我们已经看过了数组和 slice，接下来我们将看看 Go 中的另一个关键的内建数据类型：map。

map 是 Go 内置关联数据类型（在一些其他的语言中称为哈希 或者字典 ）。

```go
package main

import "fmt"

func main() {
    // 要创建一个空 map，需要使用内建函数make
    // 其形式为：make(map[key-type]val-type).
    m := make(map[string]int)

    // 使用典型的 name[key] = val 语法来设置键值对。
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
    fmt.Println("k2:", k2)   // k2: 0

    k1, prs := m["k1"]
    fmt.Println("prs:", prs) // prs: true
    fmt.Println("k1:", k1)   // k1: 7

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
k2: 0
prs: true
k1: 7
map: map[bar:2 foo:1]
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

	// The first value is the starting byte index of the rune and the second the rune itself.
	for i, c := range "go" {
		fmt.Println(i, c)
		// 0 103
		// 1 111
	}

	for i, r := range "啊哈" {
		fmt.Println(i, r, string(r))
		// 0 21834 啊
		//3 21704 哈
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
0 21834 啊
3 21704 哈
```

# 函数

```go
package main

import "fmt"

// 这里是一个函数，接受两个 int 并返回它们的和，返回值为int类型
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

    // 如果你仅仅想返回值的一部分的话，你可以使用空白标识符 _。
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

Go 支持 [*匿名函数*](http://zh.wikipedia.org/wiki/匿名函数)，可以形成 [*闭包*](http://zh.wikipedia.org/wiki/闭包_(计算机科学)) 。当您想定义一个内联函数而不必命名时，匿名函数很有用。

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
    fmt.Println("initial:", i) // 1
    zeroval(i)
    fmt.Println("zeroval:", i) // 1
    
	// 通过 &i 语法来取得 i 的内存地址，例如一个变量i 的指针。
    zeroptr(&i)
    fmt.Println("zeroptr:", i) // 0

    // 指针也是可以被打印的。
    fmt.Println("pointer:", &i) // 0x42131100
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



# Strings and Runes

A Go string is a read-only slice of bytes. The language and the standard library treat strings specially - as containers of text encoded in [UTF-8](https://en.wikipedia.org/wiki/UTF-8). In other languages, strings are made of “characters”. In Go, the concept of a character is called a `rune` - it’s an integer that represents a Unicode code point. [This Go blog post](https://go.dev/blog/strings) is a good introduction to the topic.

```go
package main

import (
    "fmt"
    "unicode/utf8"
)

func main() {

    // s is a string assigned a literal value representing the word “hello” in the Thai language. 
    // Go string literals are UTF-8 encoded text.
    const s = "สวัสดี"

    // Since strings are equivalent to []byte, this will produce the length of the raw bytes stored within.
    fmt.Println("Len:", len(s)) // Len: 18

    // Indexing into a string produces the raw byte values at each index. 
    // This loop generates the hex values of all the bytes that constitute the code points in s.
    // 对字符串进行索引会在每个索引处生成原始字节值。
    // 此循环生成构成 s 中代码点的所有字节的十六进制值。
    for i := 0; i < len(s); i++ {
        fmt.Printf("%x ", s[i]) // e0 b8 aa e0 b8 a7 e0 b8 b1 e0 b8 aa e0 b8 94 e0 b8 b5 
    }
    fmt.Println()

    // To count how many runes are in a string, we can use the utf8 package. 
    // Note that the run-time of RuneCountInString dependes on the size of the string, because it has to decode each UTF-8 rune sequentially. 
    // Some Thai characters are represented by multiple UTF-8 code points, so the result of this count may be surprising.
    fmt.Println("Rune count:", utf8.RuneCountInString(s)) // Rune count: 6

    // A range loop handles strings specially and decodes each rune along with its offset in the string.
    for idx, runeValue := range s {
        fmt.Printf("%#U starts at %d\n", runeValue, idx)
        // U+0E2A 'ส' starts at 0
	   // U+0E27 'ว' starts at 3
	   // U+0E31 'ั' starts at 6
       // U+0E2A 'ส' starts at 9
       // U+0E14 'ด' starts at 12
       // U+0E35 'ี' starts at 15
    }

    // We can achieve the same iteration by using the utf8.DecodeRuneInString function explicitly.
    fmt.Println("\nUsing DecodeRuneInString")
    for i, w := 0, 0; i < len(s); i += w {
        runeValue, width := utf8.DecodeRuneInString(s[i:])
        fmt.Printf("%#U starts at %d\n", runeValue, i)
        w = width

        // This demonstrates passing a rune value to a function.
        examineRune(runeValue)
    }
    // Using DecodeRuneInString
    // U+0E2A 'ส' starts at 0
    // found so sua
    // U+0E27 'ว' starts at 3
    // U+0E31 'ั' starts at 6
    // U+0E2A 'ส' starts at 9
    // found so sua
    // U+0E14 'ด' starts at 12
    // U+0E35 'ี' starts at 15
}

func examineRune(r rune) {

    // Values enclosed in single quotes are rune literals. 
    // We can compare a rune value to a rune literal directly.
    if r == 't' {
        fmt.Println("found tee")
    } else if r == 'ส' {
        fmt.Println("found so sua")
    }
}
```



```bash
$ go run strings-and-runes.go
Len: 18
e0 b8 aa e0 b8 a7 e0 b8 b1 e0 b8 aa e0 b8 94 e0 b8 b5 
Rune count: 6
U+0E2A 'ส' starts at 0
U+0E27 'ว' starts at 3
U+0E31 'ั' starts at 6
U+0E2A 'ส' starts at 9
U+0E14 'ด' starts at 12
U+0E35 'ี' starts at 15

Using DecodeRuneInString
U+0E2A 'ส' starts at 0
found so sua
U+0E27 'ว' starts at 3
U+0E31 'ั' starts at 6
U+0E2A 'ส' starts at 9
found so sua
U+0E14 'ด' starts at 12
U+0E35 'ี' starts at 15
```



# 结构体

Go 的*结构体(struct)* 是带类型的字段(fields)集合。 这在组织数据时非常有用。

```go
package main

import "fmt"

// This person struct type has name and age fields.
type person struct {
    name string
    age  int
}

// newPerson constructs a new person struct with the given name.
func newPerson(name string) *person {

    // You can safely return a pointer to local variable as a local variable will survive the scope of the function.
    p := person{name: name}
    p.age = 42
    return &p
}

func main() {

    // This syntax creates a new struct.
    fmt.Println(person{"Bob", 20}) // {Bob 20}

    // You can name the fields when initializing a struct.
    fmt.Println(person{name: "Alice", age: 30}) // {Alice 30}

    // Omitted fields will be zero-valued.
    fmt.Println(person{name: "Fred"}) // {Fred 0}

    // An & prefix yields a pointer to the struct.
    fmt.Println(&person{name: "Ann", age: 40}) // &{Ann 40}

    // It’s idiomatic to encapsulate new struct creation in constructor functions
    fmt.Println(newPerson("Jon")) // &{Jon 42}

    s := person{name: "Sean", age: 50}
    // Access struct fields with a dot.
    fmt.Println(s.name) // Sean

    // You can also use dots with struct pointers - the pointers are automatically dereferenced.
    sp := &s
    fmt.Println(sp.age) // 50

    // Structs are mutable.
    sp.age = 51
    fmt.Println(sp.age) // 51
}
```


```bash
$ go run structs.go
{Bob 20}
{Alice 30}
{Fred 0}
&{Ann 40}
&{Jon 42}
Sean
50
51
```

结构体中可以包含函数类型的字段：

```go
package main

import "fmt"

type person struct {
	name string
	age  int
	say  func() // 结构体中可以包含函数类型的字段
}

func newPerson(name string) *person {

	p := person{name: name}
	p.age = 42
	p.say = func() { fmt.Println("yoyo") }
	return &p
}

func main() {
	p := person{name: "Peter", age: 12, say: func() { fmt.Println("haha") }}
	fmt.Println(p) // {Peter 12 0xede5c0}
	p.say()        // haha

	p1 := newPerson("Neo")
	fmt.Println(p1) // &{Neo 42 0xaae480}
	p1.say()        // yoyo

}
```

```bash
$ go run demo.go
{Peter 12 0xede5c0}
haha
&{Neo 42 0xaae480}
yoyo
```



# 方法

Go 支持为结构体类型定义 ***方法** (methods)*  。

```go
package main

import "fmt"

type rect struct {
    width, height int
}

// area 是一个方法，该方法拥有一个 *rect 类型（rect类型的指针）的接收器(receiver)。
// You may want to use a pointer receiver type to avoid copying on method calls or to allow the method to mutate the receiving struct.
func (r *rect) area() int {
    return r.width * r.height
}

// 方法的接收器(receiver)类型可以被定义为值类型或者指针类型。
// 这是一个值类型接收器的例子。
func (r rect) perim() int {
    return 2*r.width + 2*r.height
}

func main() {
    r := rect{width: 10, height: 5}
    
	// 这里我们调用上面为结构体定义的两个方法。
    fmt.Println("area: ", r.area()) // area:  50
    fmt.Println("perim:", r.perim()) // perim: 30

    // Go automatically handles conversion between values and pointers for method calls.
    rp := &r
    fmt.Println("area: ", rp.area()) // area:  50
    fmt.Println("perim:", rp.perim()) // perim: 30
    
    // 与上面等价，注意，需要在取地址符外加上括号
    fmt.Println("(&r).area(): ", (&r).area()) // (&r).area():  50
    fmt.Println("(&r).perim():", (&r).perim()) // (&r).perim(): 30
}
```

```bash
$ go run methods.go
area:  50
perim: 30
area:  50
perim: 30
(&r).area():  50
(&r).perim(): 30
```

使用接收器类型为指针的方法，既可以避免方法调用时的值拷贝，也允许方法修改接收到的结构体中的字段。

```go
package main

import "fmt"

type person struct {
	name string
	age  int
}

func (p person) SayHello() {
	fmt.Println("my name is", p.name)
}

func (p person) ChangeName(new string) {
	p.name = new
}

func (p *person) ChangeNameInPlace(new string) {
	p.name = new
}

func main() {
	p := person{name: "Peter", age: 12}
	pp := &p

	fmt.Println(p)  // {Peter 12}
	fmt.Println(pp) // &{Peter 12}

	// 在调用方法时，Go会自动处理值和指针的转换；
	// 你可以直接使用结构体的值来调用方法，也可以使用结构体的指针来调用方法；
	p.SayHello()  // my name is Peter
	pp.SayHello() // my name is Peter

	// 不管是使用值调用方法还是使用结构体指针调用方法，都无法直接修改结构体中的字段；
	p.ChangeName("Guy")
	fmt.Println(p.name) // Peter
	pp.ChangeName("Goodman")
	fmt.Println(p.name) // Peter

	// 只有在方法的接收器类型是指针时，方法才能修改接收到的结构体的字段。
	p.ChangeNameInPlace("Guy")
	fmt.Println(p.name) // Guy
	pp.ChangeNameInPlace("Goodman")
	fmt.Println(p.name) // Goodman
}
```

```bash
$ go run .\demo1.go
{Peter 12}
&{Peter 12}
my name is Peter
my name is Peter
Peter
Peter
Guy
Goodman
```



# 接口 (Interfaces)

方法签名的集合叫做 ***接口** (Interfaces)*。

```go
package main

import (
    "fmt"
    "math"
)

// 这是一个几何形状的基本接口。
type geometry interface {
    area() float64
    perim() float64
}

// 在这个例子中，我们将为 rect 和 circle 实现该接口。
type rect struct {
    width, height float64
}

type circle struct {
    radius float64
}

// 要在 Go 中实现一个接口，我们只需要实现接口中的所有方法。
// 为 rect 实现 geometry 接口。
func (r rect) area() float64 {
    return r.width * r.height
}

func (r rect) perim() float64 {
    return 2*r.width + 2*r.height
}

// 为 circle 实现 geometry 接口。
func (c circle) area() float64 {
    return math.Pi * c.radius * c.radius
}

func (c circle) perim() float64 {
    return 2 * math.Pi * c.radius
}

// 如果一个变量实现了某个接口，我们就可以调用指定接口中的方法。 
// 这有一个通用的 measure 函数，我们可以通过它来使用所有的 geometry。
func measure(g geometry) {
    fmt.Println("g :", g) 
	fmt.Println("g.area() :", g.area())
	fmt.Println("g.perim() :", g.perim())
}

func main() {
    r := rect{width: 3, height: 4}
    c := circle{radius: 5}

    // 结构体类型 circle 和 rect 都实现了 geometry 接口， 所以我们可以将其实例作为 measure 的参数。
    measure(r)
    measure(c)
}
```

```bash
$ go run interfaces.go
g : {3 4}
g.area() : 12                
g.perim() : 14               
g : {5}                      
g.area() : 78.53981633974483 
g.perim() : 31.41592653589793
```

```go
package main

import "fmt"

type person struct {
	name string
	age  int
}

type dog struct {
	name string
	age  int
}

// 定义animal接口
type animal interface {
	eat()
	speak()
}

// person类型实现animal接口
func (p person) eat() {
	fmt.Printf("%s eat KFC.\n", p.name)
}

func (p person) speak() {
	fmt.Printf("my name is %s.\n", p.name)
}

// dog类型实现animal接口
func (d dog) eat() {
	fmt.Println("dog eat meat.")
}

func (d dog) speak() {
	fmt.Println("Woof woof~")
}

// call函数接受一个animal接口
func call(a animal) {
	a.eat()
	a.speak()
}

func main() {
	p := person{name: "Peter", age: 12}
	d := dog{name: "Dog", age: 1}

	call(p)
	call(d)
}
```

```bash
Peter eat KFC.
my name is Peter.
dog eat meat.
Woof woof~
```



# Struct Embedding

Go supports *embedding* of structs and interfaces to express a more seamless *composition* of types. This is not to be confused with `//go:embed` which is a go directive introduced in Go version 1.16+ to embed files and folders into the application binary.

```go
package main

import "fmt"

// 定义一个 base 结构体
type base struct {
    num int
}

// 定义一个base类型的方法
func (b base) describe() string {
    return fmt.Sprintf("base with num=%v", b.num)
}

// 定义一个 container 结构体，在 container 中嵌入 base。
type container struct {
    base // “嵌入”看起来像是没有名称的字段
    str string
}

func main() {
    // When creating structs with literals, we have to initialize the embedding explicitly; here the embedded type serves as the field name.
    // 当使用字面量创建结构体时，必须显式地初始化“嵌入”。
    co := container{
        base: base{ // 这里为被嵌入的类型提供一个字段名 base。
            num: 1,
        },
        str: "some name",
    }
    
	// We can access the base’s fields directly on co, e.g. co.num.
    fmt.Printf("co={num: %v, str: %v}\n", co.num, co.str) // co={num: 1, str: some name}

    // Alternatively, we can spell out the full path using the embedded type name.
    fmt.Println("co.base.num: ", co.base.num) // co.base.num:  1
    
	// Since container embeds base, the methods of base also become methods of a container. Here we invoke a method that was embedded from base directly on co.
    fmt.Println("co.describe(): ", co.describe()) // co.describe():  base with num=1  
    fmt.Println("co.base.describe(): ", co.base.describe()) // co.base.describe():  base with num=1
    
    type describer interface {
        describe() string
    }
    
	// Embedding structs with methods may be used to bestow interface implementations onto other structs. Here we see that a container now implements the describer interface because it embeds base.
    // 使用方法嵌入结构体可用于将接口实现赋予其他结构体。
    // 因为 container 嵌入了 base, 所以 container 也就实现了 describer 接口
    var d describer = co
    fmt.Println("d.describe(): ", d.describe()) // d.describe():  base with num=1
}
```

```bash
$ go run embedding.go
co={num: 1, str: some name}
co.base.num:  1
co.describe():  base with num=1
co.base.describe():  base with num=1
d.describe():  base with num=1
```

# Generics

Starting with version 1.18, Go has added support for *generics*, also known as *type parameters*.

As an example of a generic function, `MapKeys` takes a map of any type and returns a slice of its keys. This function has two type parameters - `K` and `V`; `K` has the `comparable` *constraint*, meaning that we can compare values of this type with the `==` and `!=` operators. This is required for map keys in Go. `V` has the `any` constraint, meaning that it’s not restricted in any way (`any` is an alias for `interface{}`).

```go
package main

import "fmt"

func MapKeys[K comparable, V any](m map[K]V) []K {
    r := make([]K, 0, len(m))
    for k := range m {
        r = append(r, k)
    }
    return r
}

// As an example of a generic type, List is a singly-linked list with values of any type.
type List[T any] struct {
    head, tail *element[T]
}

type element[T any] struct {
    next *element[T]
    val  T
}

// We can define methods on generic types just like we do on regular types, but we have to keep the type parameters in place. 
// The type is List[T], not List.
func (lst *List[T]) Push(v T) {
    if lst.tail == nil {
        lst.head = &element[T]{val: v}
        lst.tail = lst.head
    } else {
        lst.tail.next = &element[T]{val: v}
        lst.tail = lst.tail.next
    }
}

func (lst *List[T]) GetAll() []T {
    var elems []T
    for e := lst.head; e != nil; e = e.next {
        elems = append(elems, e.val)
    }
    return elems
}

func main() {
    var m = map[int]string{1: "2", 2: "4", 4: "8"}

    // When invoking generic functions, we can often rely on type inference. 
    // Note that we don’t have to specify the types for K and V when calling MapKeys - the compiler infers them automatically.
    fmt.Println("keys m:", MapKeys(m))

    // though we could also specify them explicitly.
    _ = MapKeys[int, string](m)

    lst := List[int]{}
    lst.Push(10)
    lst.Push(13)
    lst.Push(23)
    fmt.Println("list:", lst.GetAll())
}
```

```bash
keys: [4 1 2]
list: [10 13 23]
```

# 错误

In Go it’s idiomatic to communicate errors via an explicit, separate return value. This contrasts with the exceptions used in languages like Java and Ruby and the overloaded single result / error value sometimes used in C. Go’s approach makes it easy to see which functions return errors and to handle them using the same language constructs employed for any other, non-error tasks.

符合 Go 语言习惯的做法是使用一个独立、明确的返回值来传递错误信息。 这与 Java、Ruby 使用的异常（exception） 以及在 C 语言中有时用到的重载 (overloaded) 的单返回/错误值有着明显的不同。 Go 语言的处理方式能清楚的知道哪个函数返回了错误，并使用跟其他（无异常处理的）语言类似的方式来处理错误。

```go
package main

import (
    "errors"
    "fmt"
)

// 按照惯例，错误通常是最后一个返回值并且是 error 类型，它是一个内建的接口。
func f1(arg int) (int, error) {
    if arg == 42 {
        // errors.New 使用给定的错误信息构造一个基本的 error 值。
        return -1, errors.New("can't work with 42")
    }
    
	// A nil value in the error position indicates that there was no error.
    // 返回的错误值为 nil 代表没有错误。
    return arg + 3, nil
}

// It’s possible to use custom types as errors by implementing the Error() method on them. Here’s a variant on the example above that uses a custom type to explicitly represent an argument error.
// 你还可以通过实现 Error() 方法来自定义 error 类型。 
// 这里是上面示例的一个变体，使用自定义错误类型来表示参数错误。
type argError struct {
    arg  int
    prob string
}

func (e *argError) Error() string {
    return fmt.Sprintf("%d - %s", e.arg, e.prob)
}

func f2(arg int) (int, error) {
    if arg == 42 {
		// In this case we use &argError syntax to build a new struct, supplying values for the two fields arg and prob.
        // 在这个例子中，我们使用 &argError 语法来建立一个新的结构体， 并提供了 arg 和 prob 两个字段的值。
        return -1, &argError{arg, "can't work with it"} 
        // return -1, &(argError{arg, "can't work with it"})
    }
    return arg + 3, nil
}

func main() {
	// The two loops below test out each of our error-returning functions.
    // 下面的两个循环测试了我们的每个错误返回函数
    for _, i := range []int{7, 42} {
        // Note that the use of an inline error check on the if line is a common idiom in Go code.
        if r, e := f1(i); e != nil { // 请注意，在 if 行上使用内联错误检查是 Go 代码中的常见习惯用法。
            fmt.Println("f1 failed:", e)
        } else {
            fmt.Println("f1 worked:", r)
        }
    }
    for _, i := range []int{7, 42} {
        if r, e := f2(i); e != nil {
            fmt.Println("f2 failed:", e)
        } else {
            fmt.Println("f2 worked:", r)
        }
    }
    
	// If you want to programmatically use the data in a custom error, you’ll need to get the error as an instance of the custom error type via type assertion.
    // 如果你想在程序中使用自定义错误类型的数据， 你需要通过类型断言来得到这个自定义错误类型的实例。
    _, e := f2(42)
    if ae, ok := e.(*argError); ok {
        fmt.Println(ae.arg)
        fmt.Println(ae.prob)
    }
}
```

```bash
$ go run errors.go
f1 worked: 10
f1 failed: can't work with 42
f2 worked: 10
f2 failed: 42 - can't work with it
42
can't work with it
```

# Go协程

*协程(goroutine)* 是轻量级的执行线程。

```go
package main

import (
    "fmt"
    "time"
)

func f(from string) {
    for i := 0; i < 3; i++ {
        fmt.Println(from, ":", i)
    }
}

func main() {
    
	// 假设我们有一个函数叫做 f(s)。 我们一般会这样 同步地 调用它
    f("direct")
	
    // 使用 go f(s) 在一个协程中调用这个函数。 这个新的 Go 协程将会 并发地 执行这个函数。
    go f("goroutine")

    // 你也可以为匿名函数启动一个协程。
    go func(msg string) {
        fmt.Println(msg)
    }("going")
	
    // 现在两个协程在独立的协程中 异步地 运行， 然后等待两个协程完成（更好的方法是使用 WaitGroup）。
    time.Sleep(time.Second)
    fmt.Println("done")
}
// 当我们运行这个程序时，首先会看到阻塞式调用的输出，然后是两个协程的交替输出。 这种交替的情况表示 Go runtime 是以并发的方式运行协程的。
```

```bash
$ go run goroutines.go
direct : 0
direct : 1
direct : 2
goroutine : 0
going
goroutine : 1
goroutine : 2
done
```



# 通道

*通道(channels)* 是连接多个协程的管道。 你可以从一个协程将值发送到通道，然后在另一个协程中接收。

```go
package main

import "fmt"

func main() {

    // 使用 make(chan val-type) 创建一个新的通道。 通道类型就是他们需要传递值的类型。
    messages := make(chan string)
    
	// 使用 channel <- 语法 发送 一个新的值到通道中。 这里我们在一个新的协程中发送 "ping" 到上面创建的 messages 通道中。
    go func() { messages <- "ping" }()
    
	// 使用 <-channel 语法从通道中 接收 一个值。 这里我们会收到在上面发送的 "ping" 消息并将其打印出来。
    msg := <-messages
    fmt.Println(msg)
}
// 我们运行程序时，通过通道， 成功的将消息 "ping" 从一个协程传送到了另一个协程中。
```

```bash
$ go run channels.go
ping
```

默认发送和接收操作是阻塞的、同步的，直到发送方和接收方都就绪。 这个特性允许我们，不使用任何其它的同步操作， 就可以在程序结尾处等待消息 "ping"。



# 通道缓冲

默认情况下，通道是 *无缓冲* 的，这意味着只有对应的接收（`<- chan`） 通道准备好接收时，才允许进行发送（`chan <-`）。 *有缓冲的通道 (*Buffered channels*)* 允许在没有对应接收者的情况下，缓存一定数量的值。

```go
package main

import "fmt"

func main() {
	// 这里我们 make 了一个字符串通道，最多允许缓存 2 个值。
    messages := make(chan string, 2)

	// 由于此通道是有缓冲的， 因此我们可以将这些值发送到通道中，而无需并发的接收。
    messages <- "buffered"
    messages <- "channel"

	// 然后我们可以正常接收这两个值。
    fmt.Println(<-messages)
    fmt.Println(<-messages)
}
```

```bash
$ go run channel-buffering.go 
buffered
channel
```



# 通道同步

我们可以使用通道来同步协程之间的执行状态。 这有一个例子，使用阻塞接收的方式，实现了等待另一个协程完成。 如果需要等待多个协程，**[WaitGroup](https://gobyexample-cn.github.io/waitgroups)** 是一个更好的选择。

```go
package main

import (
    "fmt"
    "time"
)

// 我们将要在协程中运行这个函数。 done 通道将被用于通知其他协程这个函数已经完成工作。
func worker(done chan bool) {
    fmt.Println("working...")
    time.Sleep(time.Second * 5)
    fmt.Println("done...")
    
	// 发送一个值来通知我们已经完工啦。
    done <- true
}

func main() {
	// 运行一个 worker 协程，并给予用于通知的通道。
    done := make(chan bool, 1)
    go worker(done)
	// 程序将一直阻塞，直至收到 worker 使用通道发送的通知。
    <-done // 如果你把 <- done 这行代码从程序中移除， 程序甚至可能在 worker 开始运行前就结束了。
}
```

```bash
$ go run channel-synchronization.go
working...
done...
```
