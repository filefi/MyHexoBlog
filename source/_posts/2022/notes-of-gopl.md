---
title: 《The Go Programming Language》学习笔记
date: 2022-09-30 14:24:21
updated: 2022-09-30 14:24:21
tags: [Golang]
categories: Golang
---

# 前言

## [Go语言起源](https://gopl-zh.github.io/preface.html#go语言起源)

下图展示了有哪些早期的编程语言对Go语言的设计产生了重要影响。

![img](ch0-01.png)

## [Go语言项目](https://gopl-zh.github.io/preface.html#go语言项目)

Go项目包括编程语言本身，附带了相关的工具和标准库，最后但并非代表不重要的是，关于简洁编程哲学的宣言。

Go语言的这些地方都做的还不错：

- 拥有自动垃圾回收

- 一个包系统

- 函数作为一等公民

- 词法作用域

- 系统调用接口

- 只读的UTF8字符串等

但是Go语言本身只有很少的特性：

- 没有隐式的数值转换
- 没有构造函数和析构函数
- 没有运算符重载
- 没有默认参数
- 没有继承
- ~~没有泛型~~
- 没有异常
- 没有宏
- 没有函数修饰
- 没有线程局部存储

在实践中，Go语言简洁的类型系统给程序员带来了更多的安全性和更好的运行时性能。

Go语言提供了基于`CSP`的并发特性支持。Go语言的动态栈使得轻量级线程`goroutine`的初始栈可以很小，因此，创建一个`goroutine`的代价很小，创建百万级的`goroutine`完全是可行的。

Go语言的标准库（通常被称为语言自带的电池），提供了清晰的构建模块和公共接口，包含I/O操作、文本处理、图像、密码学、网络和分布式应用程序等，并支持许多标准化的文件格式和编解码协议。

<!-- more -->

# 入门

## [Hello, World](https://gopl-zh.github.io/ch1/ch1-01.html#11-hello-world)

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, 世界") // Println 函数可以打印以空格间隔的一个或多个值，并在最后添加一个换行符
}

```

Go 是一门编译型语言，Go 语言的工具链将源代码及其依赖转换成计算机的机器指令（译注：静态编译）。

Go 语言提供的工具都通过一个单独的命令 `go` 调用，`go` 命令有一系列子命令。

`run` 子命令编译一个或多个以`.go` 结尾的源文件，链接库文件，并运行最终生成的可执行文件。

```shell
$ go run helloworld.go
Hello, 世界
```

 `build` 子命令能够编译这个程序，保存编译后的可执行文件：

```shell
$ go build helloworld.go
```

这个命令生成一个与源代码同名的可执行二进制文件，之后你可以随时运行它。

```shell
$ ./helloworld
Hello, 世界
```

### Go 源文件的组成部分

**包（package）**

Go 语言的代码通过**包**（package）组织，包类似于其它语言里的库（libraries）或者模块（modules）。一个包由位于单个目录下的一个或多个 `.go` 源代码文件组成，目录定义包的作用。

 **`package` 声明**

每个源文件都以一条 `package` 声明语句开始，这个例子里就是 `package main`，表示该文件属于哪个包，紧跟着一系列导入（import）的包，之后是存储在这个文件里的程序语句。

`main` 包比较特殊。它定义了一个独立可执行的程序，而不是一个库。在 `main` 里的 `main` *函数* 也很特殊，它是整个程序执行时的入口。`main` 函数一般调用其它包里的函数完成很多工作（如：`fmt.Println`）。

**`import` 声明**

`import` 声明必须跟在文件的 `package` 声明之后，它告诉编译器源文件需要导入哪些包。`hello world` 例子只用到了一个包，大多数程序需要导入多个包。缺少了必要的包或者导入了不需要的包，程序都无法编译通过。

Go 的标准库提供了 100 多个包，以支持常见功能，如输入、输出、排序以及文本处理。比如 `fmt` 包，就含有格式化输出、接收输入的函数。`Println` 是其中一个基础函数，可以打印以空格间隔的一个或多个值，并在最后添加一个换行符。

**程序语句**

紧跟在`import`声明之后的，则是组成程序的函数、变量、常量、类型的声明语句（分别由关键字 `func`、`var`、`const`、`type` 定义），以及其他程序语句。

Go 语言不需要在语句或者声明的末尾添加分号，除非一行上有多条语句。

Go 语言在代码格式上采取了很强硬的态度。`gofmt`工具把代码格式化为标准格式，并且 `go` 工具中的 `fmt` 子命令会对指定包，否则默认为当前目录中所有。

## 命令行参数

大多数的程序都是处理输入，产生输出；这也正是“计算”的定义。命令行参数就是最主要的输入源之一。

`os` 包以跨平台的方式，提供了一些与操作系统交互的函数和变量。程序的命令行参数可从 `os` 包的 `Args` 变量获取；`os` 包外部使用 `os.Args` 访问该变量。

`os.Args` 变量是一个字符串（string）的 *切片*（slice）。现在先把切片 `s` 当作数组元素序列，序列的长度动态变化，用 `s[i]` 访问单个元素，用 `s[m:n]` 获取子序列。序列的元素数目为 `len(s)`。和大多数编程语言类似，区间索引时，Go 语言里也采用左闭右开形式，即，区间包括第一个索引元素，不包括最后一个，比如 `a=[1,2,3,4,5]`, `a[0:3]=[1,2,3]`，不包含索引为3的元素；比如 `s[m:n]` 这个切片，`0≤m≤n≤len(s)`，包含 `n-m` 个元素。如果省略切片表达式的 `m` 或 `n`，会默认传入 `0` 或 `len(s)`，即`s[:]`等同于`s[0:len(s)]`。

`os.Args` 的第一个元素：`os.Args[0]`，是命令本身的名字；其它的元素则是程序启动时传给它的参数。

下面是 Unix 里 `echo` 命令的一份实现，`echo` 把它的命令行参数打印成一行。

```go
// Echo1 prints its command-line arguments.
package main

import (
    "fmt"
    "os"
)

func main() {
    var s, sep string
    for i := 1; i < len(os.Args); i++ {
        s += sep + os.Args[i]
        sep = " "
    }
    fmt.Println(s)
}
```

注释语句以 `//` 开头。

`var` 声明定义了两个 `string` 类型的变量 `s` 和 `sep`。变量会在声明时直接初始化。如果变量没有显式初始化，则被隐式地赋予其类型的 *零值*（zero value），数值类型是 `0`，字符串类型是空字符串 `""`。

运算符 `+=` 是赋值运算符（assignment operator），每种数值运算符或逻辑运算符，如 `+` 或 `*`，都有对应的赋值运算符。

循环索引变量 `i` 在 `for` 循环的第一部分中定义。符号 `:=` 是 *短变量声明*（short variable declaration）的一部分，这是定义一个或多个变量并根据它们的初始值为这些变量赋予适当类型的语句。

自增语句 `i++` 给 `i` 加 `1`；这和 `i+=1` 以及 `i=i+1` 都是等价的。对应的还有 `i--` 给 `i` 减 `1`。它们是语句，而不像 C 系的其它语言那样是表达式，所以 `j=i++` 非法。

Go 语言只有 `for` 循环这一种循环语句。`for` 循环有多种形式，其中一种如下所示：

```go
for initialization; condition; post {
    // zero or more statements
}
```

`for` 循环三个部分不需括号包围。大括号强制要求，左大括号必须和 *`post`* 语句在同一行。

- *`initialization`* 语句是可选的，在循环开始前执行。*`initalization`* 如果存在，必须是一条 *简单语句*（simple statement），即，短变量声明、自增语句、赋值语句或函数调用。
- `condition` 是一个布尔表达式（boolean expression），其值在每次循环迭代开始时计算。如果为 `true` 则执行循环体语句。
- `post` 语句在循环体执行结束后执行，之后再次对 `condition` 求值。`condition` 值为 `false` 时，循环结束。

for 循环的这三个部分每个都可以省略，如果省略 `initialization` 和 `post`，分号也可以省略：

```go
// a traditional "while" loop
for condition {
    // ...
}
```

如果连 `condition` 也省略了，像下面这样：

```go
// a traditional infinite loop
for {
    // ...
}
```

for 循环的另一种形式，在某种数据类型的区间（range）上遍历，如字符串或切片。以下 echo 的第二版本展示了这种形式：

```go
// Echo2 prints its command-line arguments.
package main

import (
    "fmt"
    "os"
)

func main() {
    s, sep := "", ""
    for _, arg := range os.Args[1:] {
        s += sep + arg
        sep = " "
    }
    fmt.Println(s)
}
```

每次循环迭代，`range` 产生一对值；索引以及在该索引处的元素值。这个例子不需要索引，但 `range` 的语法要求，要处理元素，必须处理索引。并且 Go 语言不允许使用无用的局部变量（local variables）。这种情况适用于 *空标识符*（blank identifier），即 `_`（也就是下划线）。*空标识符* 可用于在任何语法需要变量名但程序逻辑不需要的时候（如：在循环里）丢弃不需要的循环索引，并保留元素值。

声明一个变量有好几种方式，下面这些都等价：

```go
s := ""  // 短变量声明，最简洁，但只能用在函数内部，而不能用于包变量。
var s string // 依赖于字符串的默认初始化零值机制，被初始化为 ""。
var s = ""
var s string = ""
```

如果连接涉及的数据量很大，这种方式代价高昂。一种简单且高效的解决方案是使用 `strings` 包的 `Join` 函数：

```go
func main() {
    fmt.Println(strings.Join(os.Args[1:], " "))
}
```

## [查找重复的行](https://gopl-zh.github.io/ch1/ch1-03.html#13-查找重复的行)

本节会实现一个名为 `dup` 的程序的三个版本；灵感来自于 Unix 的 `uniq` 命令，其寻找相邻的重复行。

`dup` 的第一个版本打印标准输入中多次出现的行，以重复次数开头。该程序将引入 `if` 语句，`map` 数据类型以及 `bufio` 包。

```go
// Dup1 prints the text of each line that appears more than
// once in the standard input, preceded by its count.
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    counts := make(map[string]int)
    input := bufio.NewScanner(os.Stdin)
    for input.Scan() {
        counts[input.Text()]++
    }
    // NOTE: ignoring potential errors from input.Err()
    for line, n := range counts { // line 为 key, n 为 value
        if n > 1 {
            fmt.Printf("%d\t%s\n", n, line)
        }
    }
}
```

正如 `for` 循环一样，`if` 语句条件两边也不加括号，但是主体部分需要加。`if` 语句的 `else` 部分是可选的，在 `if` 的条件为 `false` 时执行。

**map** 存储了键/值（key/value）的集合，对集合元素，提供常数时间的存、取或测试操作。键可以是任意类型，只要其值能用 `==` 运算符比较，最常见的例子是字符串；值则可以是任意类型。这个例子中的键是字符串，值是整数。

每次 `dup` 读取一行输入，该行被当做键存入 `map`，其对应的值递增。`counts[input.Text()]++` 语句等价下面两句：

```go
line := input.Text()
counts[line] = counts[line] + 1
```

`map` 中不含某个键时不用担心，首次读到新行时，等号右边的表达式 `counts[line]` 的值将被计算为其类型的零值，对于 `int` 即 `0`。

为了打印结果，我们使用了基于 `range` 的循环，以迭代 `counts`。与迭代 slice 类似，迭代 map 每次得到两个结果：键和键的值。

> **注意： map 的迭代顺序被有意设计成不确定的、随机的，每次运行迭代，其顺序都会变化。**

`bufio` 包使处理输入和输出方便又高效。`Scanner` 类型是该包最有用的特性之一，它读取输入并将其拆成行或单词；通常是处理行形式的输入最简单的方法。

程序使用短变量声明创建 `bufio.Scanner` 类型的变量 `input`。

```go
input := bufio.NewScanner(os.Stdin)
```

该变量从程序的标准输入中读取内容。每次调用 `input.Scan()`，即读入下一行，并移除行末的换行符；读取的内容可以调用 `input.Text()` 得到。`Scan` 函数在读到一行时返回 `true`，不再有输入时返回 `false`。

类似于 C 或其它语言里的 `printf` 函数，`fmt.Printf` 函数对一些表达式产生格式化输出。该函数的首个参数是个格式字符串，指定后续参数被如何格式化。各个参数的格式取决于“转换字符”（conversion character），形式为百分号后跟一个字母。

`Printf` 有一大堆这种转换字符，Go程序员称之为*动词（verb）*。下表展示了常用的几个：

```text
%d          十进制整数
%x, %o, %b  十六进制，八进制，二进制整数。
%f, %g, %e  浮点数： 3.141593 3.141592653589793 3.141593e+00
%t          布尔：true或false
%c          字符（rune） (Unicode码点)
%s          字符串
%q          带双引号的字符串"abc"或带单引号的字符'c'
%v          变量的自然形式（natural format）
%T          变量的类型
%%          字面上的百分号标志（无操作数）
```

默认情况下，`Printf` 不会换行，除非格式字符串中存在换行符`\n`。

按照惯例，以字母 `f` 结尾的格式化函数，如 `log.Printf` 和 `fmt.Errorf`，都采用 `fmt.Printf` 的格式化准则。而以 `ln` 结尾的格式化函数，则遵循 `Println` 的方式，以跟 `%v` 差不多的方式格式化参数，并在最后添加一个换行符。（译注：后缀 `f` 指 `format`，`ln` 指 `line`。）

`dup` 程序的下个版本读取标准输入或是使用 `os.Open` 打开各个具名文件，并操作它们。

```go
// Dup2 prints the count and text of lines that appear more than once
// in the input.  It reads from stdin or from a list of named files.
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    counts := make(map[string]int)
    files := os.Args[1:]
    if len(files) == 0 {
        countLines(os.Stdin, counts)
    } else {
        for _, arg := range files {
            // 打开文件
            f, err := os.Open(arg) // os.Open 返回 *os.File 和 error
            if err != nil { // 如果 err 等于内置值nil（相当于其它语言里的 NULL），那么文件被成功打开。
                fmt.Fprintf(os.Stderr, "dup2: %v\n", err)
                continue
            }
            countLines(f, counts)
            f.Close() // 关闭文件f
        }
    }
    for line, n := range counts {
        if n > 1 {
            fmt.Printf("%d\t%s\n", n, line)
        }
    }
}

func countLines(f *os.File, counts map[string]int) {
    input := bufio.NewScanner(f)
    for input.Scan() {
        counts[input.Text()]++
    }
    // NOTE: ignoring potential errors from input.Err()
}
```

注意 `countLines` 函数在其声明前被调用。函数和包级别的变量（package-level entities）可以任意顺序声明，并不影响其被调用。

`map` 是一个由 `make` 函数创建的数据结构的引用。`map` 作为参数传递给某函数时，该函数接收这个引用的一份拷贝（copy，或译为副本），被调用函数对 `map` 底层数据结构的任何修改，调用者函数都可以通过持有的 `map` 引用看到。在我们的例子中，`countLines` 函数向 `counts` 插入的值，也会被 `main` 函数看到。（译注：类似于 C++ 里的引用传递，实际上指针是另一个指针了，但内部存的值指向同一块内存）

> **注意：Go 语言只有按值传递，传递的都是变量的一个副本，一个拷贝。只不过拷贝的内容，可能是非引用类型（int、string、struct等这些），这样就在函数中就无法修改原内容数据；可能是引用类型（指针、map、slice、chan等这些），这样就可以修改原内容数据。**

`dup` 的前两个版本以"流”模式读取输入，并根据需要拆分成多个行。理论上，这些程序可以处理任意数量的输入数据。下面这个版本`dup3`则是一口气把文件的数据全部读到内存中，一次分割为多行，然后处理它们：

```go
package main

import (
    "fmt"
    "io"
    "os"
    "strings"
)

func main() {
    counts := make(map[string]int)
    for _, filename := range os.Args[1:] {
        data, err := os.ReadFile(filename)
        if err != nil {
            fmt.Fprintf(os.Stderr, "dup3: %v\n", err)
            continue
        }
        for _, line := range strings.Split(string(data), "\n") {
            counts[line]++
        }
    }
    for line, n := range counts {
        if n > 1 {
            fmt.Printf("%d\t%s\n", n, line)
        }
    }
}
```

这个例子中， `os.ReadFile` 函数，其读取指定文件的全部内容。`strings.Split` 函数的作用与前文提到的 `strings.Join` 相反，把字符串分割成子串的切片。

> 包 `io/ioutil` 已被弃用， 新代码推荐使用包 `io` 和 `os`中的实现。如：`ioutil.ReadFile()`变为`os.ReadFile()`，`ioutil.ReadAll()`变为`io.ReadAll()`。

`os.ReadFile` 函数返回一个字节切片（byte slice），必须把它转换为 `string`，才能用 `strings.Split` 分割。

实现上，`bufio.Scanner`、`ioutil.ReadFile` 和 `ioutil.WriteFile` 都使用 `*os.File` 的 `Read` 和 `Write` 方法，但是，大多数程序员很少需要直接调用那些低级（lower-level）函数。像 `bufio` 和 `io/ioutil` 包中所提供的那些高级（higher-level）函数，用起来要容易点。

## [GIF 动画](https://gopl-zh.github.io/ch1/ch1-04.html#14-gif动画)

下面的程序会演示Go语言标准库里的image这个package的用法，我们会用这个包来生成一系列的bit-mapped图，然后将这些图片编码为一个GIF动画。我们生成的图形名字叫利萨如图形（Lissajous figures），这种效果是在1960年代的老电影里出现的一种视觉特效。它们是协振子在两个纬度上振动所产生的曲线，比如两个sin正弦波分别在x轴和y轴输入会产生的曲线。图1.1是这样的一个例子：

![img](ch1-01.png)

译注：要看这个程序的结果，需要将标准输出重定向到一个GIF图像文件（使用 `./lissajous > output.gif` 命令）。下面是GIF图像动画效果：

![img](ch1-01.gif)

```go
// Lissajous generates GIF animations of random Lissajous figures.
package main

import (
    "image"
    "image/color"
    "image/gif"
    "io"
    "math"
    "math/rand"
    "os"
    "time"
)

// 常量声明和变量声明在包级别，在整个包中都是可以共享的。
// 变量的字面量定义
var palette = []color.Color{color.White, color.Black}

// 常量声明
const (
    whiteIndex = 0 // first color in palette
    blackIndex = 1 // next color in palette
)

func main() {
    // The sequence of images is deterministic unless we seed
    // the pseudo-random number generator using the current time.
    // Thanks to Randall McPherson for pointing out the omission.
    rand.Seed(time.Now().UTC().UnixNano())
    lissajous(os.Stdout)
}

func lissajous(out io.Writer) {
    // 把常量声明定义在函数体内部，那么这种常量就只能在函数体内用。
    const (
        cycles  = 5     // number of complete x oscillator revolutions
        res     = 0.001 // angular resolution
        size    = 100   // image canvas covers [-size..+size]
        nframes = 64    // number of animation frames
        delay   = 8     // delay between frames in 10ms units
    )

    freq := rand.Float64() * 3.0 // relative frequency of y oscillator
    anim := gif.GIF{LoopCount: nframes}
    phase := 0.0 // phase difference
    for i := 0; i < nframes; i++ {
        rect := image.Rect(0, 0, 2*size+1, 2*size+1)
        img := image.NewPaletted(rect, palette)
        for t := 0.0; t < cycles*2*math.Pi; t += res {
            x := math.Sin(t)
            y := math.Sin(t*freq + phase)
            img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5),
                blackIndex)
        }
        phase += 0.1
        anim.Delay = append(anim.Delay, delay)
        anim.Image = append(anim.Image, img)
    }
    gif.EncodeAll(out, &anim) // NOTE: ignoring encoding errors
}

```

main函数调用lissajous函数，用它来向标准输出流打印信息，所以下面这个命令会像图1.1中产生一个GIF动画。

```shell
$ go build gopl.io/ch1/lissajous
$ ./lissajous >out.gif
```

## [获取 URL](https://gopl-zh.github.io/ch1/ch1-05.html#15-获取url)

利用Go语言的`net`包和其他建立在`net`包基础之上的一系列包，可以更简单地用网络收发信息，还可以建立更底层的网络连接，编写服务器程序。在这些情景下，Go语言原生的并发特性（在第八章中会介绍）显得尤其好用。

为了最简单地展示基于HTTP获取信息的方式，下面给出一个示例程序fetch，这个程序将获取对应的url，并将其源文本打印出来：

```go
// Fetch prints the content found at a URL.
package main

import (
    "fmt"
    "io"
    "net/http"
    "os"
)

func main() {
    for _, url := range os.Args[1:] {
        resp, err := http.Get(url) // 创建HTTP请求的函数
        if err != nil {
            fmt.Fprintf(os.Stderr, "fetch: %v\n", err)
            os.Exit(1)
        }
        b, err := io.ReadAll(resp.Body) // io.ReadAll函数从response中读取全部内容到b
        resp.Body.Close() // 关闭resp的Body流，防止资源泄露
        if err != nil {
            fmt.Fprintf(os.Stderr, "fetch: reading %s: %v\n", url, err)
            os.Exit(1)
        }
        fmt.Printf("%s", b)
    }
}
```

```go
$ go build gopl.io/ch1/fetch
$ ./fetch http://gopl.io
<html>
<head>
<title>The Go Programming Language</title>title>
...

```

HTTP请求如果失败了的话，会得到下面这样的结果：

```shell
$ ./fetch http://bad.gopl.io
fetch: Get http://bad.gopl.io: dial tcp: lookup bad.gopl.io: no such host
```

译注：在大天朝的网络环境下很容易重现这种错误，下面是Windows下运行得到的错误信息：

```shell
$ go run main.go http://gopl.io
fetch: Get http://gopl.io: dial tcp: lookup gopl.io: getaddrinfow: No such host is known.
```

无论哪种失败原因，我们的程序都用了`os.Exit`函数来终止进程，并且返回一个status错误码，其值为1。

## [并发获取多个 URL](https://gopl-zh.github.io/ch1/ch1-06.html#16-并发获取多个url)

Go语言最有意思并且最新奇的特性就是对并发编程的支持。这里我们只浅尝辄止地来体验一下Go语言里的goroutine和channel。

下面的例子fetchall，和前面小节的fetch程序所要做的工作基本一致，fetchall的特别之处在于它会同时去获取所有的URL，所以这个程序的总执行时间不会超过执行时间最长的那一个任务，前面的fetch程序执行时间则是所有任务执行时间之和。fetchall程序只会打印获取的内容大小和经过的时间，不会像之前那样打印获取的内容。

```go
// Fetchall fetches URLs in parallel and reports their times and sizes.
package main

import (
    "fmt"
    "io"
    "net/http"
    "os"
    "time"
)

func main() {
    start := time.Now()
    ch := make(chan string) // 创建一个传递string类型参数的channel
    for _, url := range os.Args[1:] {
        go fetch(url, ch) // start a goroutine
    }
    // 注意 for range 不一定非得使用短变量声明接收迭代中每一项的值
    for range os.Args[1:] {
        fmt.Println(<-ch) // receive from channel ch
    }
    fmt.Printf("%.2fs elapsed\n", time.Since(start).Seconds())
}

func fetch(url string, ch chan<- string) {
    start := time.Now()
    resp, err := http.Get(url)
    if err != nil {
        ch <- fmt.Sprint(err) // send to channel ch
        return
    }
    // 因为我们需要这个方法返回的字节数，但是又不想要其内容。
    // io.Copy把响应的Body拷贝到io.Discard输出流中丢弃
    nbytes, err := io.Copy(io.Discard, resp.Body)
    resp.Body.Close() // don't leak resources
    if err != nil {
        ch <- fmt.Sprintf("while reading %s: %v", url, err)
        return
    }
    secs := time.Since(start).Seconds()
    ch <- fmt.Sprintf("%.2fs  %7d  %s", secs, nbytes, url)
}
```

下面使用fetchall来请求几个地址：

```shell
$ go build gopl.io/ch1/fetchall
$ ./fetchall https://golang.org http://gopl.io https://godoc.org
0.14s     6852  https://godoc.org
0.16s     7261  https://golang.org
0.48s     2475  http://gopl.io
0.48s elapsed
```

goroutine是一种函数的并发执行方式，而channel是用来在goroutine之间进行参数传递。main函数本身也运行在一个goroutine中，而go function则表示创建一个新的goroutine，并在这个新的goroutine中执行这个函数。

当一个goroutine尝试在一个channel上做send或者receive操作时，这个goroutine会阻塞在调用处，直到另一个goroutine从这个channel里接收或者写入值，这样两个goroutine才会继续执行channel操作之后的逻辑。在这个例子中，每一个fetch函数在执行时都会往channel里发送一个值（`ch <- expression`），主函数负责接收这些值（`<-ch`）。

这个程序中我们用main函数来完整地处理/接收所有fetch函数传回的字符串，可以避免因为有两个goroutine同时完成而使得其输出交错在一起的危险。

## [Web服务](https://gopl-zh.github.io/ch1/ch1-07.html#17-web服务)

在本节中，我们会展示一个微型服务器，这个服务器的功能是返回当前用户正在访问的URL。比如用户访问的是 http://localhost:8000/hello ，那么响应是 `URL.Path = "hello"`。

```go
// Server1 is a minimal "echo" server.
package main

import (
    "fmt"
    "log"
    "net/http"
)

func main() {
    http.HandleFunc("/", handler) // each request calls handler
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

// handler echoes the Path component of the request URL r.
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "URL.Path = %q\n", r.URL.Path)
}
```

让我们在后台运行这个服务程序。如果你的操作系统是Mac OS X或者Linux，那么在运行命令的末尾加上一个&符号，即可让程序简单地跑在后台，windows下可以在另外一个命令行窗口去运行这个程序。

```shell
$ go run src/gopl.io/ch1/server1/main.go &
```

现在可以通过命令行来发送客户端请求了：

```shell
$ go build gopl.io/ch1/fetch
$ ./fetch http://localhost:8000
URL.Path = "/"
$ ./fetch http://localhost:8000/help
URL.Path = "/help"
```

还可以直接在浏览器里访问这个URL，然后得到返回结果，如图1.2：

![img](notes-of-gopl/ch1-02.png)

在这个服务的基础上叠加特性是很容易的。一种比较实用的修改是为访问的url添加某种状态。比如，下面这个版本输出了同样的内容，但是会对请求的次数进行计算，访问`/count`这个URL返回访问的次数。

```go
// Server2 is a minimal "echo" and counter server.
package main

import (
    "fmt"
    "log"
    "net/http"
    "sync"
)

var mu sync.Mutex
var count int

func main() {
    // 如果请求pattern是以/结尾，那么所有以该url为前缀的url都会被这条规则匹配。
    http.HandleFunc("/", handler)
    http.HandleFunc("/count", counter) // 对/count这个url的请求会调用到counter这个函数
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

// handler echoes the Path component of the requested URL.
func handler(w http.ResponseWriter, r *http.Request) {
    mu.Lock()
    count++
    mu.Unlock()
    fmt.Fprintf(w, "URL.Path = %q\n", r.URL.Path)
}

// counter echoes the number of calls so far.
func counter(w http.ResponseWriter, r *http.Request) {
    mu.Lock()
    fmt.Fprintf(w, "Count %d\n", count)
    mu.Unlock()
}
```

在这些代码的背后，服务器每一次接收请求处理时都会另起一个goroutine，这样服务器就可以同一时间处理多个请求。然而在并发情况下，假如真的有两个请求同一时刻去更新`count`，那么这个值可能并不会被正确地增加；这个程序可能会引发一个严重的bug：竞态条件（参见9.1）。为了避免这个问题，我们必须保证每次修改变量的最多只能有一个goroutine，这也就是代码里的`mu.Lock()`和`mu.Unlock()`调用将修改`count`的所有行为包在中间的目的。

下面是一个更为丰富的例子，handler函数会把请求的http头和请求的form数据都打印出来，这样可以使检查和调试这个服务更为方便：

```go
// handler echoes the HTTP request.
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "%s %s %s\n", r.Method, r.URL, r.Proto)
    for k, v := range r.Header {
        fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
    }
    fmt.Fprintf(w, "Host = %q\n", r.Host)
    fmt.Fprintf(w, "RemoteAddr = %q\n", r.RemoteAddr)
    if err := r.ParseForm(); err != nil {
        log.Print(err)
    }
    for k, v := range r.Form {
        fmt.Fprintf(w, "Form[%q] = %q\n", k, v)
    }
}
```

我们用`http.Request`这个struct里的字段来输出下面这样的内容：

```
GET /?q=query HTTP/1.1
Header["Accept-Encoding"] = ["gzip, deflate, sdch"]
Header["Accept-Language"] = ["en-US,en;q=0.8"]
Header["Connection"] = ["keep-alive"]
Header["Accept"] = ["text/html,application/xhtml+xml,application/xml;..."]
Header["User-Agent"] = ["Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_5)..."]
Host = "localhost:8000"
RemoteAddr = "127.0.0.1:59911"
Form["q"] = ["query"]
```

可以看到这里的 `ParseForm` 被嵌套在了if语句中。Go语言允许这样的一个简单的语句结果作为局部的变量声明出现在 if 语句的最前面，这一点对错误处理很有用处。等同于下面这样写：

```go
err := r.ParseForm()
if err != nil {
    log.Print(err)
}
```

用 if 和`ParseForm`结合可以让代码更加简单，并且可以限制`err`变量的作用域。

在这些程序中，我们看到了很多不同的类型被输出到标准输出流中。比如前面的fetch程序，把HTTP的响应数据拷贝到了os.Stdout，lissajous程序里我们输出的是一个文件。fetchall程序则完全忽略到了HTTP的响应Body，只是计算了一下响应Body的大小，这个程序中把响应Body拷贝到了`io.Discard`。在本节的web服务器程序中则是用`fmt.Fprintf`直接写到了`http.ResponseWriter`中。

尽管三种具体的实现流程并不太一样，但它们都实现`io.Writer`接口，即当它们被调用需要一个标准流输出时都可以满足。

让我们简单地将这里的web服务器和之前写的 lissajous 函数结合起来，这样GIF动画可以被写到HTTP的客户端，而不是之前的标准输出流。只要在web服务器的代码里加入下面这几行。

```Go
handler := func(w http.ResponseWriter, r *http.Request) {
    lissajous(w)
}
http.HandleFunc("/", handler)
```

或者另一种等价形式：

```Go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    lissajous(w)
})
```

`HandleFunc` 函数的第二个参数是一个函数的字面值，也就是一个在使用时定义的匿名函数。

做完这些修改之后，在浏览器里访问 http://localhost:8000 。每次你载入这个页面都可以看到一个像图1.3那样的动画。

![img](notes-of-gopl/ch1-03.png)



## 本章要点

**控制流：** 这里是一个简单的switch的例子：

```go
switch coinflip() {
case "heads":
    heads++
case "tails":
    tails++
default:
    fmt.Println("landed on edge!")
}
```

在翻转硬币的时候，例子里的`coinflip`函数返回几种不同的结果，每一个case都会对应一个返回结果，这里需要注意，Go语言并不需要显式地在每一个case后写break，语言默认执行完case后的逻辑语句会自动退出。

Go语言里的switch还可以不带操作对象，可以直接罗列多种条件，像其它语言里面的多个if else一样。这种形式叫做 **无tag switch** (tagless switch)；这和`switch true`是等价的。switch不带操作对象时默认用true值代替，然后将每个case的表达式和`true`值进行比较。像for和if控制语句一样，switch也可以紧跟一个简短的变量声明，一个自增表达式、赋值语句，或者一个函数调用。下面是一个例子：

```go
func Signum(x int) int {
    switch {
    case x > 0:
        return +1
    default:
        return 0
    case x < 0:
        return -1
    }
}
```

break和continue语句会改变控制流。和其它语言中的break和continue一样，break会中断当前的循环，并开始执行循环之后的内容，而continue会跳过当前循环，并开始执行下一次循环。这两个语句除了可以控制for循环，还可以用来控制switch和select语句。如果我们想跳过的是更外层的循环的话，我们可以在相应的位置加上label，这样break和continue就可以根据我们的想法来continue和break任意循环。这看起来甚至有点像goto语句的作用了。当然，一般程序员也不会用到这种操作。这两种行为更多地被用到机器生成的代码中。

**命名类型：** 类型声明使得我们可以很方便地给一个特殊类型一个名字。因为struct类型声明通常非常地长，所以我们总要给这种struct取一个名字。本章中就有这样一个例子，二维点类型：

```go
type Point struct {
    X, Y int
}
var p Point
```

**指针：** Go语言提供了指针。指针是一种直接存储了变量的内存地址的数据类型。在其它语言中，比如C语言，指针操作是完全不受约束的。在另外一些语言中，指针一般被处理为“引用”，除了到处传递这些指针之外，并不能对这些指针做太多事情。Go语言在这两种范围中取了一种平衡。指针是可见的内存地址，`&`操作符可以返回一个变量的内存地址，并且`*`操作符可以获取指针指向的变量内容，但是在Go语言里没有指针运算，也就是不能像c语言里可以对指针进行加或减操作。

**方法和接口：** **方法**是和命名类型关联的一类函数。Go语言里比较特殊的是方法可以被关联到任意一种命名类型。**接口**是一种抽象类型，这种类型可以让我们以同样的方式来处理不同的固有类型，不用关心它们的具体实现，而只需要关注它们提供的方法。

**包（packages）：** Go语言提供了一些很好用的package，并且这些package是可以扩展的。

在你开始写一个新程序之前，最好先去检查一下是不是已经有了现成的库可以帮助你更高效地完成这件事情。你可以在 https://golang.org/pkg 和 https://godoc.org 中找到标准库和社区写的package。godoc 这个工具可以让你直接在本地命令行阅读标准库的文档。比如下面这个例子。

```
$ go doc http.ListenAndServe
package http // import "net/http"
func ListenAndServe(addr string, handler Handler) error
    ListenAndServe listens on the TCP network address addr and then
    calls Serve with handler to handle requests on incoming connections.
...
```

**注释：** 我们之前已经提到过了在源文件的开头写的注释是这个源文件的文档。在每一个函数之前写一个说明函数行为的注释也是一个好习惯。这些惯例很重要，因为这些内容会被像godoc这样的工具检测到，并且在执行命令时显示这些注释。

多行注释可以用 `/* ... */` 来包裹，和其它大多数语言一样。在文件一开头的注释一般都是这种形式，或者一大段的解释性的注释文字也会被这符号包住，来避免每一行都需要加`//`。在注释中`//`和`/*`是没什么意义的，所以不要在注释中再嵌入注释。



# 程序结构





















