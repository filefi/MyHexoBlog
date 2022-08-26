---
title: 从《A Tour of Go》到《Effective Go》
date: 2022-08-26 09:36:11
updated: 2022-08-26 09:36:11
tags: [Golang, Programming]
categories: Golang
---

> 本文是Golang的学习笔记，内容涵盖 [A Tour of Go](https://go.dev/tour/) 官方文档 [Effective Go](https://go.dev/doc/effective_go) 以及 [官方翻译](https://go-zh.org/doc/effective_go.html)

<!-- more -->

# 引言

# 格式化

# 注释

# 命名

## 包名

## 获取器

## 接口名

## 驼峰记法

# 分号

# 控制结构

## if 条件语句

## 重新生命与再次赋值

## for 循环

## switch

## 类型选择

# 函数

## 多返回值 ([Multiple return values](https://go.dev/doc/effective_go#multiple-returns))

## 可命名结果形参 ([Named result parameters](https://go.dev/doc/effective_go#named-results))

## [Defer](https://go.dev/doc/effective_go#defer)

# 数据

### 零值 ([The zero value](https://go.dev/ref/spec#The_zero_value))

When storage is allocated for a [variable](https://go.dev/ref/spec#Variables), either through a declaration or a call of `new`, or when a new value is created, either through a composite literal or a call of `make`, and no explicit initialization is provided, the variable or value is given a default value. Each element of such a variable or value is set to the *zero value* for its type: `false` for booleans, `0` for numeric types, `""` for strings, and `nil` for pointers, functions, interfaces, slices, channels, and maps. This initialization is done recursively, so for instance each element of an array of structs will have its fields zeroed if no value is specified.

当通过声明或 new 调用为变量分配存储空间时，或通过复合文字或 make 调用创建新值时，且未提供显式初始化，则变量或值会被给定一个默认值。变量或值的每个元素都被设置为其类型的零值：对于布尔值，其零值是`false`；对于数值类型，其零值是`0`；对于字符串，其零值是`""`；对于指针、函数、接口、切片、信道 (channels)和映射，其零值是`nil`。初始化操作是递归完成的。例如，如果没有指定，则结构体数组的每个元素都被置为其字段的零值。

以下两种声明是等价的：
```go
var i int
var i int = 0
```

例如，下面`i1`虽然声明时未赋值，但自动被赋零值。

```go
var i1 int
var i2 int = 0
fmt.Println(i1 == i2) // true
```

又或者下面这个结构体T，通过调用`new`声明变量t：

```go
type T struct { i int; f float64; next *T }
t := new(T)
```

这个结构体变量`t`的成员字段零值如下：

```go
t.i == 0 // true
t.f == 0.0 // true
t.next == nil // true
```

直接声明结构体T的变量`t`也是同样的：

```go
var t T
fmt.Println(t.i == 0) // true
fmt.Println(t.f == 0.0) // true
fmt.Println(t.next == nil) // true
```

## `new` 分配

 `new`是个用来分配内存的内建函数， 但与其它语言中的同名函数不同，它不会**初始化**内存，只会将内存**置零**。 也就是说，`new(T)` 会为类型为 `T` 的新项分配已置零的内存空间， 并返回它的地址，也就是一个类型为 `*T` 的值。用Go的术语来说，它返回一个指针， 该指针指向新分配的，类型为 `T` 的零值。

Since the memory returned by `new` is zeroed, it's helpful to arrange when designing your data structures that the zero value of each type can be used without further initialization. This means a user of the data structure can create one with `new` and get right to work. For example, the documentation for `bytes.Buffer` states that "the zero value for `Buffer` is an empty buffer ready to use." Similarly, `sync.Mutex` does not have an explicit constructor or `Init` method. Instead, the zero value for a `sync.Mutex` is defined to be an unlocked mutex.

既然 `new` 返回的内存已置零，那么每种类型的零值不必进一步初始化就可以使用了。这在设计数据结构时是非常有用的。这意味着该数据结构的使用者只需用 `new` 创建一个新的对象就能正常工作。例如，`bytes.Buffer` 的文档中提到“零值的 `Buffer` 就是已准备就绪的缓冲区。" 同样，`sync.Mutex` 并没有显式的构造函数或 `Init` 方法， 而是零值的 `sync.Mutex` 就已经被定义为已解锁的互斥锁了。

The zero-value-is-useful property works transitively. Consider this type declaration.

```go
type SyncedBuffer struct {
	lock    sync.Mutex
	buffer  bytes.Buffer
}
```

Values of type `SyncedBuffer` are also ready to use immediately upon allocation or just declaration. In the next snippet, both `p` and `v` will work correctly without further arrangement.

```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

## 构造函数与复合字面量 ([Constructors and composite literals](https://go.dev/doc/effective_go#constructors_and_composite_literals))

有时零值还不够好，这时就需要一个初始化构造函数，如来自 `os` 包中的这段代码所示。

```go
func NewFile(fd int, name string) *File {
	if fd < 0 {
		return nil
	}
	f := new(File)
	f.fd = fd
	f.name = name
	f.dirinfo = nil
	f.nepipe = 0
	return f
}
```

这里显得代码过于冗长。我们可通过**复合字面量**来简化它， 该表达式在每次求值时都会创建新的实例。

```go
func NewFile(fd int, name string) *File {
	if fd < 0 {
		return nil
	}
	f := File{fd, name, nil, 0}
	return &f
}
```

每当获取一个复合字面的地址时，都将为一个新的实例分配内存， 因此我们可以将上面的最后两行代码合并：

```go
return &File{fd, name, nil, 0}
```

复合字面的字段必须按顺序全部列出。但如果以 **字段**`:`**值** 对的形式明确地标出元素，初始化字段时就可以按任何顺序出现，未给出的字段值将赋予零值。 因此，我们可以用如下形式：

```go
return &File{fd: fd, name: name}
```

As a limiting case, if a composite literal contains no fields at all, it creates a zero value for the type. The expressions `new(File)` and `&File{}` are equivalent.

作为一种极端情况，若复合字面不包括任何字段，它将创建该类型的零值。表达式 `new(File)` 和 `&File{}` 是等价的。

## `make` 分配

内建函数 `make(T, args)` 的目的不同于 `new(T)`。它只用于创建切片、映射和信道 (Channel)，并返回类型为 `T`（而非 `*T`）的一个**已初始化** （而非**置零**）的值。出现这种用差异的原因在于，这三种类型本质上为引用数据类型，它们在使用前必须初始化。 例如，切片是一个具有三项内容的描述符，包含一个指向（数组内部）数据的指针、长度以及容量， 在这三项被初始化之前，该切片为 `nil`。对于切片、映射和信道，`make` 用于初始化其内部的数据结构并准备好将要使用的值。例如，

```go
make([]int, 10, 100)
```

会分配一个具有100个 `int` 的数组空间，接着创建一个长度为10， 容量为100并指向该数组中前10个元素的切片结构。（生成切片时，其容量可以省略，更多信息见切片一节。） 与此相反，`new([]int)` 会返回一个指向新分配的，已置零的切片结构， 即一个指向 `nil` 切片值的指针。

下面的例子阐明了 `new` 和 `make` 之间的区别：

```go
var p *[]int = new([]int)       // 分配切片结构；*p == nil；几乎没什么用
var v  []int = make([]int, 100) // 切片 v 现在引用了一个具有 100 个 int 元素的新数组

// 没必要的复杂：
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// 习惯用法：
v := make([]int, 100)
```

请记住，`make` 只适用于映射、切片和信道且不返回指针。若要获得明确的指针， 请使用 `new` 分配内存。

## new和make的区别



## 数组

类型 `[n]T` 表示拥有 `n` 个 `T` 类型的值的数组。

表达式：

```go
var a [10]int
```

会将变量 `a` 声明为拥有 10 个整数的数组。

这是一个数组字面量 (literal)：

```go
[3]bool{true, true, false}
```

在详细规划内存布局时，数组是非常有用的，有时还能避免过多的内存分配， 但它们主要用作切片的构件。

以下为数组在Go和C中的主要区别。在Go中，

- 数组是值。将一个数组赋予另一个数组会复制其所有元素。
- 特别地，若将某个数组传入某个函数，它将接收到该数组的一份**副本**而非指针。
- 数组的大小是其类型的一部分，因此数组不能改变大小。类型 `[10]int` 和 `[20]int` 是不同的。

The value property can be useful but also expensive; if you want C-like behavior and efficiency, you can pass a pointer to the array.

数组为值的属性很有用，但代价高昂；若你想要C那样的行为和效率，你可以传递一个指向该数组的指针。

```go
func Sum(a *[3]float64) (sum float64) {
	for _, v := range *a {
		sum += v
	}
	return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // 注意显式的取址操作
```

但这并不是Go的习惯用法。使用切片，而不是数组。

## 切片

每个数组的大小都是固定的。切片通过对数组进行封装，为数据序列提供了更通用、强大而方便的接口。 除了矩阵变换这类需要明确维度的情况外，Go中的大部分数组编程都是通过切片来完成的。

类型 `[]T` 表示一个元素类型为 `T` 的切片。

切片通过两个下标来界定，即一个上界和一个下界，二者以冒号分隔：

```go
a[low : high]
```

它会选择一个半开区间，包括第一个元素，但排除最后一个元素。

以下表达式创建了一个切片，它包含 `a` 中下标从 1 到 3 的元素：

```go
a[1:4]
```

切片保存了对底层数组的引用，若你将某个切片赋予另一个切片，它们会引用同一个数组。 更改切片的元素会修改其底层数组中对应的元素。与它共享底层数组的切片都会观测到这些修改。

```go
names := [4]string{
    "John",
    "Paul",
    "George",
    "Ringo",
}
fmt.Println(names) // [John Paul George Ringo]

a := names[0:2]
b := names[1:3]
fmt.Println(a, b) // [John Paul] [Paul George]

b[0] = "XXX"
fmt.Println(a, b) // [John XXX] [XXX George]
fmt.Println(names) // [John XXX George Ringo]
```

### 切片的零值

切片的零值是 `nil`。`nil` 切片的长度和容量为 0 且没有底层数组。

### 切片字面量 (Slice Literals)

切片字面量类似于没有长度的数组文法。下面这样则会创建一个和上面相同的数组，然后构建一个引用了它的切片：

```go
[]bool{true, true, false}
```

### 用 make 创建切片

切片可以用内建函数 `make` 来创建，这也是你创建动态数组的方式。

`make` 函数会分配一个元素为零值的数组并返回一个引用了它的切片：

```go
a := make([]int, 5)  // len(a)=5
```

要指定它的容量，需向 `make` 传入第三个参数：

```go
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```

### 切片的默认上下界

切片下界的默认值为 `0`，上界则是该切片的长度。

对于数组

```go
var a [10]int
```

来说，以下切片是等价的：

```go
a[0:10]
a[:10]
a[0:]
a[:]
```

### 切片作为函数参数

若某个函数将一个切片作为参数传入，则它对该切片元素的修改对调用者而言同样可见， 这可以理解为传递了底层数组的指针。因此，`Read` 函数可接受一个切片实参 而非一个指针和一个计数；切片的长度决定了可读取数据的上限。以下为 `os` 包中 `File` 类型的 `Read` 方法签名:

```go
func (file *File) Read(buf []byte) (n int, err error)
```

该方法返回读取的字节数和一个错误值（若有的话）。若要从更大的缓冲区 `buf` 中读取前32个字节，只需对其进行**切片**即可。

```go
	n, err := f.Read(buf[0:32])
```

这种切片的方法常用且高效。

### 切片的长度与容量

切片拥有 **长度** 和 **容量**。切片的**长度**就是它所包含的元素个数。切片的**容量**是从它的第一个元素开始数，到其底层数组元素末尾的个数。

切片 `s` 的长度和容量可通过内建函数 `len(s)` 和 `cap(s)` 来获取。

你可以通过重新切片来扩展一个切片，给它提供足够的容量。

```go
s := []int{2, 3, 5, 7, 11, 13}
fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s) // len=6 cap=6 [2 3 5 7 11 13]

// 截取切片使其长度为 0
s = s[:0]
fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s) // len=0 cap=6 []

// 拓展其长度
s = s[:4]
fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s) // len=4 cap=6 [2 3 5 7]

// 舍弃前两个值
s = s[2:]
fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s) // len=2 cap=4 [5 7]
```

只要切片不超出底层数组的限制，它的长度就是可变的，只需将它赋给其自身的切片即可。  以下是将数据追加到切片的函数。若数据超出其容量，则会重新分配该切片。返回值即为所得的切片。 该函数中所使用的 `len` 和 `cap` 在应用于 `nil` 切片时是合法的，它会返回0.

```go
func Append(slice, data []byte) []byte {
	l := len(slice)
	if l + len(data) > cap(slice) {  // 重新分配
		// 为了未来的增长，分配双倍。
		newSlice := make([]byte, (l+len(data))*2)
		// copy 函数是预声明的，且可用于任何切片类型。
		copy(newSlice, slice)
		slice = newSlice
	}
	slice = slice[0:l+len(data)]
	for i, c := range data {
		slice[l+i] = c
	}
	return slice
}
```

最终我们必须返回切片，因为尽管 `Append` 可修改 `slice` 的元素，但切片自身（其运行时数据结构包含指针、长度和容量）是通过值传递的。

为切片追加新的元素是种常用的操作，因此有专门的内建函数 `append`：

-  `append` 的第一个参数 `s` 是一个元素类型为 `T` 的切片，其余类型为 `T` 的值将会追加到该切片的末尾。
- `append` 的结果是一个包含原切片所有元素加上新添加元素的切片。

```go
func append(s []T, vs ...T) []T
```

当 `s` 的底层数组太小，不足以容纳所有给定的值时，它就会分配一个更大的数组。返回的切片会指向这个新分配的数组。

## 二维切片

Go的数组和切片都是一维的。要创建等价的二维数组或切片，就必须定义一个数组的数组， 或切片的切片，就像这样：

```go
type Transform [3][3]float64  // 一个 3x3 的数组，其实是包含多个数组的一个数组。
type LinesOfText [][]byte     // 包含多个字节切片的一个切片。
```

由于切片长度是可变的，因此其内部可能拥有多个不同长度的切片。在我们的 `LinesOfText` 例子中，这是种常见的情况：每行都有其自己的长度。

```go
text := LinesOfText{
	[]byte("Now is the time"),
	[]byte("for all good gophers"),
	[]byte("to bring some fun to the party."),
}
```

有时必须分配一个二维数组，例如在处理像素的扫描行时，这种情况就会发生。 我们有两种方式来达到这个目的。一种就是独立地分配每一个切片；而另一种就是只分配一个数组， 将各个切片都指向它。采用哪种方式取决于你的应用。若切片会增长或收缩， 就应该通过独立分配来避免覆盖下一行；若不会，用单次分配来构造对象会更加高效。 以下是这两种方法的大概代码，仅供参考。首先是一次一行的：

```go
// 分配顶层切片。
picture := make([][]uint8, YSize) // 每 y 个单元一行。
// 遍历行，为每一行都分配切片
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

现在是一次分配，对行进行切片：

```go
// 分配顶层切片，和前面一样。
picture := make([][]uint8, YSize) // 每 y 个单元一行。
// 分配一个大的切片来保存所有像素
pixels := make([]uint8, XSize*YSize) // 拥有类型 []uint8，尽管图片是 [][]uint8.
// 遍历行，从剩余像素切片的前面切出每行来。
for i := range picture {
	picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

## 映射

映射是方便而强大的内建数据结构，它可以关联不同类型的值。其键可以是任何相等性操作符支持的类型， 如整数、浮点数、复数、字符串、指针、接口（只要其动态类型支持相等性判断）、结构以及数组。 切片不能用作映射键，因为它们的相等性是未定义。与切片一样，映射也是引用类型。 若将映射传入函数中，并更改了该映射的内容，则此修改对调用者同样可见。

映射的零值为 `nil` 。`nil` 映射既没有键，也不能添加键。

`make` 函数可以返回给定类型的映射，并将其初始化。

映射可使用一般的复合字面语法进行构建，其键-值对使用逗号分隔，因此可在初始化时很容易构建它们。

```go
var timeZone = map[string]int{
	"UTC":  0*60*60,
	"EST": -5*60*60,
	"CST": -6*60*60,
	"MST": -7*60*60,
	"PST": -8*60*60,
}
```

赋值和获取映射值的语法类似于数组，不同的是映射的索引不必为整数。

```go
offset := timeZone["EST"]
```

若试图通过映射中不存在的键来取值，就会返回与该映射中项的类型对应的零值。 例如，若某个映射包含整数，当查找一个不存在的键时会返回 `0`。 

可以通过一个值类型为 `bool` 的映射来实现**集合**。将该映射中的项置为 `true` 可将该值放入集合中，此后通过简单的索引操作即可判断是否存在。

```go
attended := map[string]bool{
	"Ann": true,
	"Joe": true,
	...
}

if attended[person] { // 若某人不在此映射中，则为 false
	fmt.Println(person, "正在开会")
}
```

有时你需要区分某项是不存在还是其值为零值。如对于一个值本应为零的 `"UTC"` 条目，也可能是由于不存在该项而得到零值。你可以使用多重赋值的形式来分辨这种情况。

```go
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

显然，我们可称之为“逗号 ok”惯用法。在下面的例子中，若 `tz` 存在， `seconds` 就会被赋予适当的值，且 `ok` 会被置为 `true`； 若不存在，`seconds` 则会被置为零，而 `ok` 会被置为 `false`。

```go
func offset(tz string) int {
	if seconds, ok := timeZone[tz]; ok {
		return seconds
	}
	log.Println("unknown time zone:", tz)
	return 0
}
```

若仅需判断映射中是否存在某项而不关心实际的值，可使用[空白标识符](https://go-zh.org/doc/effective_go.html#空白) （`_`）来代替该值的一般变量。

```go
_, present := timeZone[tz]
```

要删除映射中的某项，可使用内建函数 `delete`，它以映射及要被删除的键为实参。 即便对应的键不在该映射中，此操作也是安全的。

```go
delete(timeZone, "PDT")  // 现在用标准时间
```

映射的值可以是结构体：

```go
type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}
```

若顶级类型只是一个类型名，你可以在字面量的元素中省略它：

```go
type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}
```

## 打印

Go采用的格式化打印风格和C的 `printf` 族类似，但却更加丰富而通用。 这些函数位于 `fmt` 包中，且函数名首字母均为大写：如 `fmt.Printf`、`fmt.Fprintf`，`fmt.Sprintf` 等。 字符串函数（`Sprintf` 等）会返回一个字符串，而非填充给定的缓冲区。

你无需提供一个格式字符串。每个 `Printf`、`Fprintf` 和 `Sprintf` 都分别对应另外的函数，如 `Print` 与 `Println`。 这些函数并不接受格式字符串，而是为每个实参生成一种默认格式。`Println` 系列的函数还会在实参中插入空格，并在输出时追加一个换行符，而 `Print` 版本仅在操作数两侧都没有字符串时才添加空白。以下示例中各行产生的输出都是一样的。

```go
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```

`fmt.Fprint` 一类的格式化打印函数可接受任何实现了 `io.Writer` 接口的对象作为第一个实参；变量`os.Stdout` 与 `os.Stderr` 都是人们熟知的例子。

从这里开始，就与C有些不同了。首先，像 `%d` 这样的数值格式并不接受表示符号或大小的标记， 打印例程会根据实参的类型来决定这些属性。

```go
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
```

将打印

```go
18446744073709551615 ffffffffffffffff; -1 -1
```

若你只想要默认的转换，如使用十进制的整数，你可以使用通用的格式 `%v`（对应“值”）；其结果与 `Print` 和 `Println` 的输出完全相同。此外，这种格式还能打印**任意**值，甚至包括数组、结构体和映射。 以下是打印上一节中定义的时区映射的语句。

```go
fmt.Printf("%v\n", timeZone)  // 或只用 fmt.Println(timeZone)
```

这会输出

```go
map[CST:-21600 PST:-28800 EST:-18000 UTC:0 MST:-25200]
```

当然，映射中的键可能按任意顺序输出。当打印结构体时，改进的格式 `%+v` 会为结构体的每个字段添上字段名，而另一种格式 `%#v` 将完全按照Go的语法打印值。

```go
type T struct {
	a int
	b float64
	c string
}
t := &T{ 7, -2.35, "abc\tdef" }
fmt.Printf("%v\n", t)
fmt.Printf("%+v\n", t)
fmt.Printf("%#v\n", t)
fmt.Printf("%#v\n", timeZone)
```

将打印

```go
&{7 -2.35 abc   def}
&{a:7 b:-2.35 c:abc     def}
&main.T{a:7, b:-2.35, c:"abc\tdef"}
map[string] int{"CST":-21600, "PST":-28800, "EST":-18000, "UTC":0, "MST":-25200}
```

（请注意其中的&符号）当遇到 `string` 或 `[]byte` 值时， 可使用 `%q` 产生带引号的字符串；而格式 `%#q` 会尽可能使用反引号。 （`%q` 格式也可用于整数和符文，它会产生一个带单引号的符文常量。） 此外，`%x` 还可用于字符串、字节数组以及整数，并生成一个很长的十六进制字符串， 而带空格的格式（`% x`）还会在字节之间插入空格。

另一种实用的格式是 `%T`，它会打印某个值的**类型**.

```go
fmt.Printf("%T\n", timeZone)
```

会打印

```go
map[string] int
```

若你想控制自定义类型的默认格式，只需为该类型定义一个具有 `String() string` 签名的方法。对于我们简单的类型 `T`，可进行如下操作。

```go
func (t *T) String() string {
	return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)
```

会打印出如下格式：

```go
7/-2.35/"abc\tdef"
```

（如果你需要像指向 `T` 的指针那样打印类型 `T` 的**值**， `String` 的接收者就必须是值类型的；上面的例子中接收者是一个指针， 因为这对结构来说更高效而通用。更多详情见[指针vs.值接收者](https://go-zh.org/doc/effective_go.html#指针vs值)一节.）

我们的 `String` 方法也可调用 `Sprintf`， 因为打印例程可以完全重入并按这种方式封装。不过要理解这种方式，还有一个重要的细节： 请勿通过调用 `Sprintf` 来构造 `String` 方法，因为它会无限递归你的 `String` 方法。

```go
type MyString string

func (m MyString) String() string {
	return fmt.Sprintf("MyString=%s", m) // 错误：会无限递归
}
```

要解决这个问题也很简单：将该实参转换为基本的字符串类型，它没有这个方法。

```go
type MyString string
func (m MyString) String() string {
	return fmt.Sprintf("MyString=%s", string(m)) // 可以：注意转换
}
```

在[初始化](https://go-zh.org/doc/effective_go.html#初始化)一节中，我们将看到避免这种递归的另一种技术。

另一种打印技术就是将打印例程的实参直接传入另一个这样的例程。`Printf` 的签名为其最后的实参使用了 `...interface{}` 类型，这样格式的后面就能出现任意数量，任意类型的形参了。

```go
func Printf(format string, v ...interface{}) (n int, err error) {
```

在 `Printf` 函数中，`v` 看起来更像是 `[]interface{}` 类型的变量，但如果将它传递到另一个变参函数中，它就像是常规实参列表了。 以下是我们之前用过的 `log.Println` 的实现。它直接将其实参传递给 `fmt.Sprintln` 进行实际的格式化。

```go
// Println 通过 fmt.Println 的方式将日志打印到标准记录器。
func Println(v ...interface{}) {
	std.Output(2, fmt.Sprintln(v...))  // Output 接受形参 (int, string)
}
```

在该 `Sprintln` 嵌套调用中，我们将 `...` 写在 `v` 之后来告诉编译器将 `v` 视作一个实参列表，否则它会将 `v` 当做单一的切片实参来传递。

还有很多关于打印知识点没有提及。详情请参阅 `godoc` 对 `fmt` 包的说明文档。

顺便一提，`...` 形参可指定具体的类型，例如从整数列表中选出最小值的函数 `min`，其形参可为 `...int` 类型。

```go
func Min(a ...int) int {
	min := int(^uint(0) >> 1)  // 最大的 int
	for _, i := range a {
		if i < min {
			min = i
		}
	}
	return min
}
```

## 追加

现在我们要对内建函数 `append` 的设计进行补充说明。大致来说,`append` 函数的签名像这样：

```go
func append(slice []T, 元素 ...T) []T
```

其中的 *T* 为任意给定类型的占位符。实际上，你无法在Go中编写一个类型 `T` 由调用者决定的函数。这也就是为何 `append` 为内建函数的原因：它需要编译器的支持。

`append` 会在切片末尾追加元素并返回结果。我们必须返回结果， 原因与我们手写的 `Append` 一样，即底层数组可能会被改变。以下简单的例子

```go
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
```

将打印 `[1 2 3 4 5 6]`。因此 `append` 有点像 `Printf` 那样，可接受任意数量的实参。

但如果我们要像 `Append` 那样将一个切片追加到另一个切片中呢？ 很简单：在调用的地方使用 `...`，就像我们在上面调用 `Output` 那样。以下代码片段的输出与上一个相同。

```go
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
```

如果没有 `...`，它就会由于类型错误而无法编译，因为 `y` 不是 `int` 类型的。

