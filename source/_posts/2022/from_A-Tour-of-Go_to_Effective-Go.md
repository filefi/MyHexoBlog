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

## 零值 ([The zero value](https://go.dev/ref/spec#The_zero_value))

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

`new` 的特点：

- 分配内存空间
- 对变量和值置零值
- 返回指针
- `new(Type)` 等价于 `&File{Type}`

`make` 的特点：

- 只用于创建切片、映射和信道 (Channel)
- 对变量和值初始化
- 返回值

```go
var a []int
fmt.Println(a)        // []
fmt.Println(a == nil) // true

b := new([]int)
fmt.Println(b)         // &[]
fmt.Println(b == nil)  // false
fmt.Println(*b == nil) // true

c := make([]int, 3)
fmt.Println(c)        // [0 0 0]
fmt.Println(c == nil) // false

fmt.Printf("%T %T %T", a, b, c) // []int *[]int []int
```



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


# 初始化

尽管从表面上看，Go 的初始化过程与 C 或 C++ 并没有太大不同，但它确实更为强大。 在初始化过程中，不仅可以构建复杂的结构，还能正确处理不同包对象间的初始化顺序。

## 常量

Go 中的常量就是不变量。它们在编译时创建，即便它们可能是函数中定义的局部变量。 常量只能是数字、字符（符文）、字符串或布尔值。由于编译时的限制， 定义它们的表达式必须也是可被编译器求值的常量表达式。例如 `1<<3` 就是一个常量表达式，而 `math.Sin(math.Pi/4)` 则不是，因为对 `math.Sin` 的函数调用在运行时才会发生。

在 Go 中，枚举常量使用枚举器 `iota` 创建。由于 `iota` 可为表达式的一部分，而表达式可以被隐式地重复，这样也就更容易构建复杂的值的集合了。

```go
type ByteSize float64

const (
    // 通过赋予空白标识符来忽略第一个值
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

由于可将 `String` 之类的方法附加在用户定义的类型上， 因此它就为打印时自动格式化任意值提供了可能性，即便是作为一个通用类型的一部分。 尽管你常常会看到这种技术应用于结构体，但它对于像 `ByteSize` 之类的浮点数标量等类型也是有用的。

```go
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```

表达式 `YB` 会打印出 `1.00YB`，而 `ByteSize(1e13)` 则会打印出 `9.09`。

在这里用 `Sprintf` 实现 `ByteSize` 的 `String` 方法很安全（不会无限递归），这倒不是因为类型转换，而是它以 `%f` 调用了 `Sprintf`，它并不是一种字符串格式：`Sprintf` 只会在它需要字符串时才调用 `String` 方法，而 `%f` 需要一个浮点数值。

## 变量

变量的初始化与常量类似，但其初始值也可以是在运行时才被计算的一般表达式。

```go
var (
	home   = os.Getenv("HOME")
	user   = os.Getenv("USER")
	gopath = os.Getenv("GOPATH")
)
```

## `init` 函数

最后，每个源文件都可以通过定义自己的无参数 `init` 函数来设置一些必要的状态。 （其实每个文件都可以拥有多个 `init` 函数。）而它的结束就意味着初始化结束： 只有该包中的所有变量声明都通过它们的初始化器求值后 `init` 才会被调用， 而那些 `init` 只有在所有已导入的包都被初始化后才会被求值。

除了那些不能被表示成声明的初始化外，`init` 函数还常被用在程序真正开始执行前，检验或校正程序的状态。

```go
func init() {
	if user == "" {
		log.Fatal("$USER not set")
	}
	if home == "" {
		home = "/home/" + user
	}
	if gopath == "" {
		gopath = home + "/go"
	}
	// gopath 可通过命令行中的 --gopath 标记覆盖掉。
	flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

### [详解 Go 语言中的 init () 函数](https://learnku.com/go/t/47178)

> 原文作者：Summer
> 转自链接：https://learnku.com/go/t/47178
> 原文地址：[https://developpaper.com/detailed-explan...](https://developpaper.com/detailed-explanation-of-init-function-in-go-language/)
> 译文地址：https://learnku.com/go/t/47178
> 版权声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请保留以上作者信息和原文链接。

初始化每个包后，会自动执行 init（）函数，并且执行优先级高于主函数的执行优先级。init 函数通常用于：

- 变量初始化
- 检查 / 修复状态
- 注册器
- 运行计算

**包初始化**

为了使用导入的程序包，必须首先对其进行初始化。初始化始终在单个线程中执行，并且以程序包依赖关系的顺序执行。这由 Golang 的运行时系统控制，如下图所示：

- 初始化导入的包（递归导入）
- 计算并为块中声明的变量分配初始值
- 在包中执行初始化函数

![](ejLR69f443.png)

```go
package main
import "fmt"
var _ int64=s()
func init(){
  fmt.Println("init function --->")
}
func s() int64{
  fmt.Println("function s() --->")
  return 1
}
func main(){
  fmt.Println("main --->")
}
```

执行结果为：

```go
function s() —>
init function —>
main —>
```

即使程序包被多次导入，初始化也只需要一次。

**特性**

`init` 函数不需要传入参数，也不需要返回任何值。与 `main` 相比，`init` 没有声明，因此无法引用。

```go
package main
import "fmt"
func init(){
  fmt.Println("init")
}
func main(){
  init()
}
```
编译上述函数 `undefined：init` 时发生错误。

每个源文件可以包含一个以上的 `init` 函数，请记住，写在每个源文件中的 “行进方式” 只能包含一个 `init` 函数，这有点不同，因此进行下一个验证。

```go
package main
import "fmt"
func init(){
  fmt.Println("init 1")
}
func init(){
  fmt.Println("init2")
}
func main(){
  fmt.Println("main")
}

/* 执行结果:
init1
init2
main 
*/
```
从上面的示例中，您可以看到每个源文件可以包含多个 `init` 函数。

初始化函数的一个常见示例是设置初始表达式的值。

```go
var precomputed=[20]float64{}
func init(){
  var current float64=1
  precomputed[0]=current
  for i:=1;i<len(precomputed);i++{
    precomputed[i]=precomputed[i-1]*1.2
  }
}
```

因为不可能在上面的代码 (这是一条语句) 中将 for 循环用作预先计算的值，所以可以使用 `init` 函数来解决此问题。

**Go 套件汇入规则的副作用**

Go 非常严格，不允许引用未使用的软件包。但是有时您引用包只是为了调用 `init` 函数进行一些初始化。空标识符 (即下划线) 的目的是解决此问题。

```go
import _ "image/png"
```



# 方法

Go 没有类。不过你可以为结构体类型定义方法。

方法就是一类带特殊的 **接收者** 参数的函数。

方法接收者在它自己的参数列表内，位于 `func` 关键字和方法名之间。

在此例中，`Abs` 方法拥有一个名为 `v`，类型为 `Vertex` 的接收者。

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}
```

 **方法即函数**

记住：方法只是个带接收者参数的函数。

现在这个 `Abs` 的写法就是个正常的函数，功能并没有什么变化。

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(Abs(v))
}
```

你也可以为非结构体类型声明方法。

在此例中，我们看到了一个带 `Abs` 方法的数值类型 `MyFloat`。

你只能为在同一包内定义的类型的接收者声明方法，而不能为其它包内定义的类型（包括 `int` 之类的内建类型）的接收者声明方法。

（译注：就是接收者的类型定义和方法声明必须在同一包内；不能为内建类型声明方法。）

```go
package main

import (
	"fmt"
	"math"
)

type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

func main() {
	f := MyFloat(-math.Sqrt2)
	fmt.Println(f.Abs())
}
```

## 指针接收者

你可以为指针接收者声明 **方法** 。

这意味着对于某类型 `T`，接收者的类型可以用 `*T` 的文法。（此外，`T` 不能是像 `*int` 这样的指针。）

例如，这里为 `*Vertex` 定义了 `Scale` 方法。

**指针接收者的方法可以修改接收者指向的值（就像 `Scale` 在这做的）。由于方法经常需要修改它的接收者，指针接收者比值接收者更常用。**

若使用值接收者，那么 `Scale` 方法会对原始 `Vertex` 值的副本进行操作。（对于函数的其它参数也是如此。）`Scale` 方法必须用指针接受者来更改 `main` 函数中声明的 `Vertex` 的值。

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}
```

现在我们要把 `Abs` 和 `Scale` 方法重写为函数。

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func Scale(v Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	Scale(v, 10)
	fmt.Println(Abs(v))
}

```

## 方法与指针重定向

比较前两个程序，你大概会注意到带指针参数的函数必须接受一个指针：

```go
var v Vertex
ScaleFunc(v, 5)  // 编译错误！
ScaleFunc(&v, 5) // OK
```

而以指针为接收者的方法被调用时，接收者既能为值又能为指针：

```go
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
```

对于语句 `v.Scale(5)`，即便 `v` 是个值而非指针，带指针接收者的方法也能被直接调用。 **也就是说，由于 `Scale` 方法有一个指针接收者，为方便起见，Go 会将语句 `v.Scale(5)` 解释为 `(&v).Scale(5)`。**

同样的事情也发生在相反的方向。

接受一个值作为参数的函数必须接受一个指定类型的值：

```go
var v Vertex
fmt.Println(AbsFunc(v))  // OK
fmt.Println(AbsFunc(&v)) // 编译错误！
```

而以值为接收者的方法被调用时，接收者既能为值又能为指针：

```go
var v Vertex
fmt.Println(v.Abs()) // OK
p := &v
fmt.Println(p.Abs()) // OK
```

**这种情况下，方法调用 `p.Abs()` 会被解释为 `(*p).Abs()`。**

## 选择值或指针作为接收者

使用指针接收者的原因有二：

- 方法能够修改其接收者指向的值。
- 这样可以避免在每次调用方法时复制该值。若值的类型为大型结构体时，这样做会更加高效。

在本例中，`Scale` 和 `Abs` 接收者的类型为 `*Vertex`，即便 `Abs` 并不需要修改其接收者。

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := &Vertex{3, 4}
	fmt.Printf("Before scaling: %+v, Abs: %v\n", v, v.Abs())
	v.Scale(5)
	fmt.Printf("After scaling: %+v, Abs: %v\n", v, v.Abs())
}
```

通常来说，所有给定类型的方法都应该有值或指针接收者，但并不应该二者混用。

## 指针 vs. 值

正如 `ByteSize` 那样，我们可以为任何已命名的类型（除了指针或接口）定义方法； 接收者可不必为结构体。

在之前讨论切片时，我们编写了一个 `Append` 函数。 我们也可将其定义为切片的方法。为此，我们首先要声明一个已命名的类型来绑定该方法， 然后使该方法的接收者成为该类型的值。

```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
	// 主体和前面相同。
}
```

我们仍然需要该方法返回更新后的切片。为了消除这种不便，我们可通过重新定义该方法， 将一个指向 `ByteSlice` 的**指针**作为该方法的接收者， 这样该方法就能重写调用者提供的切片了。

```go
func (p *ByteSlice) Append(data []byte) {
	slice := *p
	// 主体和前面相同，但没有 return。
	*p = slice
}
```

若我们将函数修改为与标准 `Write` 类似的方法，就像这样，

```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
	slice := *p
	// 依旧和前面相同。
	*p = slice
	return len(data), nil
}
```

那么类型 `*ByteSlice` 就满足了标准的 `io.Writer` 接口，这将非常实用。 例如，我们可以通过打印将内容写入。

```go
	var b ByteSlice
	fmt.Fprintf(&b, "This hour has %d days\n", 7)
```

我们将 `ByteSlice` 的地址传入，因为只有 `*ByteSlice` 才满足 `io.Writer`。以指针或值为接收者的区别在于：值方法可通过指针和值调用， 而指针方法只能通过指针来调用。

之所以会有这条规则是因为指针方法可以修改接收者；通过值调用它们会导致方法接收到该值的副本， 因此任何修改都将被丢弃，因此该语言不允许这种错误。不过有个方便的例外：**若该值是可寻址的， 那么该语言就会自动插入取址操作符来对付一般的通过值调用的指针方法。在我们的例子中，变量 `b` 是可寻址的，因此我们只需通过 `b.Write` 来调用它的 `Write` 方法，编译器会将它重写为 `(&b).Write`。**

顺便一提，在字节切片上使用 `Write` 的想法已被 `bytes.Buffer` 所实现。

# 接口与其它类型

## 接口

Go中的接口为指定对象的行为提供了一种方法：如果某样东西可以完成**这个**， 那么它就可以用在**这里**。我们已经见过许多简单的示例了；通过实现 `String` 方法，我们可以自定义打印函数，而通过 `Write` 方法，`Fprintf` 则能对任何对象产生输出。在Go代码中， 仅包含一两种方法的接口很常见，且其名称通常来自于实现它的方法， 如 `io.Writer` 就是实现了 `Write` 的一类对象。

每种类型都能实现多个接口。例如一个实现了 `sort.Interface` 接口的集合就可通过 `sort` 包中的例程进行排序。该接口包括 `Len()`、`Less(i, j int) bool` 以及 `Swap(i, j int)`，另外，该集合仍然可以有一个自定义的格式化器。 以下特意构建的例子 `Sequence` 就同时满足这两种情况。

```go
package main

import (
	"fmt"
	"sort"
)

type Sequence []int

// Methods required by sort.Interface.
// sort.Interface 所需的方法。
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Method for printing - sorts the elements before printing.
// 用于打印的方法 - 在打印前对元素进行排序。
func (s Sequence) String() string {
    sort.Sort(s)
    str := "["
    for i, elem := range s {
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}

func main() {
	seq := Sequence{7, 8, 2, 1, 3, 10, 9, 4}
	fmt.Println(seq) // [1 2 3 4 7 8 9 10]
    
    seq = []int{7, 8, 2, 1, 3, 10, 9, 4}
	fmt.Println(seq) // [7 8 2 1 3 10 9 4]
}
```

## 类型转换

`Sequence` 的 `String` 方法重新实现了 `Sprint` 为切片实现的功能。若我们在调用 `Sprint` 之前将 `Sequence` 转换为纯粹的 `[]int`，就能共享 `[]int` 已实现的功能。

```go
func (s Sequence) String() string {
	sort.Sort(s)
	return fmt.Sprint([]int(s))
}

func main() {
	seq := Sequence{7, 8, 2, 1, 3, 10, 9, 4}
	fmt.Println(seq) // [1 2 3 4 7 8 9 10]
}
```

该方法是通过类型转换技术，在 `String` 方法中安全调用 `Sprintf` 的另个一例子。若我们忽略类型名的话，这两种类型（`Sequence`和 `[]int`）其实是相同的，因此在二者之间进行转换是合法的。 转换过程并不会创建新值，它只是值暂让现有的时看起来有个新类型而已。 （还有些合法转换则会创建新值，如从整数转换为浮点数等。）

在Go程序中，为访问不同的方法集而进行类型转换的情况非常常见。 例如，我们可使用现有的 `sort.IntSlice` 类型来简化整个示例：

```go
type Sequence []int

// 用于打印的方法 - 在打印前对元素进行排序。
func (s Sequence) String() string {
	sort.IntSlice(s).Sort() // 将 s 的类型转换为 sort.IntSlice 类型，然后调用 sort.IntSlice 的 Sort 方法 
	return fmt.Sprint([]int(s))
}
```

现在，不必让 `Sequence` 实现多个接口（排序和打印）， 我们可通过将数据条目转换为多种类型（`Sequence`、`sort.IntSlice` 和 `[]int`）来使用相应的功能，每次转换都完成一部分工作。 这在实践中虽然有些不同寻常，但往往却很有效。

## 接口转换与类型断言

[类型选择](https://go-zh.org/doc/effective_go.html#类型选择) 是类型转换的一种形式：它接受一个接口，在 switch 语句中根据其判断选择对应的 case 语句， 并在某种意义上将其转换为该种类型。以下代码为 `fmt.Printf` 通过类型选择将值转换为字符串的简化版。若它已经为字符串，我们需要该接口中实际的字符串值； 若它有 `String` 方法，我们则需要调用该方法所得的结果。

```go
type Stringer interface {
	String() string
}

var value interface{} // 调用者提供的值。
switch str := value.(type) {
case string:
	return str
case Stringer:
	return str.String()
}
```

第一种情况获取具体的值，第二种将该接口转换为另一个接口。这种方式对于混合类型来说非常完美。

若我们只关心一种类型呢？若我们知道该值拥有一个 `string` 而想要提取它呢？ 只需一种情况的类型选择就行，但它需要**类型断言**。类型断言接受一个接口值， 并从中提取指定的明确类型的值。其语法借鉴自类型选择开头的子句，但它需要一个明确的类型， 而非 `type` 关键字：

```go
value.(typeName)
```

而其结果则是拥有静态类型 `typeName` 的新值。该类型必须为该接口所拥有的具体类型， 或者该值可转换成的第二种接口类型。要提取我们知道在该值中的字符串，可以这样：

```go
str := value.(string)
```

但若它所转换的值中不包含字符串，该程序就会以运行时错误崩溃。为避免这种情况， 需使用`逗号, ok`惯用测试它能安全地判断该值是否为字符串：

```go
str, ok := value.(string)
if ok {
	fmt.Printf("字符串值为 %q\n", str)
} else {
	fmt.Printf("该值非字符串\n")
}
```

若类型断言失败，`str` 将继续存在且为字符串类型，但它将拥有零值，即空字符串。

作为对能力 (capability) 的说明，这里有个 `if-else` 语句，它等价于本节开头的类型选择 switch 。

```go
if str, ok := value.(string); ok {
	return str
} else if str, ok := value.(Stringer); ok {
	return str.String()
}
```

## 通用性

若某种现有的类型仅实现了一个接口，且除此之外并无可导出的方法，则该类型本身就无需导出。 仅导出该接口能让我们更专注于其行为而非实现，其它属性不同的实现则能镜像该原始类型的行为。 这也能够避免为每个通用接口的实例重复编写文档。

在这种情况下，构造函数应当返回一个接口值而非实现的类型。例如在 `hash` 库中，`crc32.NewIEEE` 和 `adler32.New` 都返回接口类型 `hash.Hash32`。要在Go程序中用 Adler-32 算法替代 CRC-32， 只需修改构造函数调用即可，其余代码则不受算法改变的影响。

同样的方式能将 `crypto` 包中多种联系在一起的流密码算法与块密码算法分开。 `crypto/cipher` 包中的 `Block` 接口指定了块密码算法的行为， 它为单独的数据块提供加密。接着，和 `bufio` 包类似，任何实现了该接口的密码包都能被用于构造以 `Stream` 为接口表示的流密码，而无需知道块加密的细节。

`crypto/cipher` 接口看其来就像这样：

```go
type Block interface {
	BlockSize() int
	Encrypt(src, dst []byte)
	Decrypt(src, dst []byte)
}

type Stream interface {
	XORKeyStream(dst, src []byte)
}
```

这是计数器模式 CTR 流的定义，它将块加密改为流加密，注意块加密的细节已被抽象化了。

```go
// NewCTR 返回一个 Stream，其加密/解密使用计数器模式中给定的 Block 进行。
// iv 的长度必须与 Block 的块大小相同。
func NewCTR(block Block, iv []byte) Stream
```

`NewCTR` 的应用并不仅限于特定的加密算法和数据源，它适用于任何对 `Block` 接口和 `Stream` 的实现。因为它们返回接口值， 所以用其它加密模式来代替CTR只需做局部的更改。构造函数的调用过程必须被修改， 但由于其周围的代码只能将它看做 `Stream`，因此它们不会注意到其中的区别。

## 接口和方法

由于几乎任何类型都能添加方法，因此几乎任何类型都能满足一个接口。一个很直观的例子就是 `http` 包中定义的 `Handler` 接口。任何实现了 `Handler` 的对象都能够处理HTTP请求。

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

`ResponseWriter` 接口提供了对方法的访问，这些方法需要响应客户端的请求。 由于这些方法包含了标准的 `Write` 方法，因此 `http.ResponseWriter` 可用于任何 `io.Writer` 适用的场景。`Request` 结构体包含已解析的客户端请求。

为简单起见，我们假设所有的HTTP请求都是GET方法，而忽略POST方法， 这种简化不会影响处理程序的建立方式。这里有个短小却完整的处理程序实现， 它用于记录某个页面被访问的次数。

```go
// 简单的计数器服务。
type Counter struct {
	n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	ctr.n++
	fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```

（紧跟我们的主题，注意 `Fprintf` 如何能输出到 `http.ResponseWriter`。） 作为参考，这里演示了如何将这样一个服务器添加到URL树的一个节点上。

```go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

但 `Counter` 不必是结构体，一个整数就够了。 An integer is all that's needed. （接收者必须为指针，增量操作对于调用者才可见。）

```go
// 简单的计数器服务。
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	*ctr++
	fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

当页面被访问时，怎样通知你的程序去更新一些内部状态呢？为Web页面绑定个信道吧。

```go
// 每次浏览该信道都会发送一个提醒。
// （可能需要带缓冲的信道。）
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	ch <- req
	fmt.Fprint(w, "notification sent")
}
```

最后，假设我们需要输出调用服务器二进制程序时使用的实参 `/args`。 很简单，写个打印实参的函数就行了。

```go
func ArgServer() {
	fmt.Println(os.Args)
}
```

我们如何将它转换为HTTP服务器呢？我们可以将 `ArgServer` 实现为某种可忽略值的方法，不过还有种更简单的方法。 既然我们可以为除指针和接口以外的任何类型定义方法，同样也能为一个函数写一个方法。 `http` 包包含以下代码：

```go
// HandlerFunc 类型是一个适配器，它允许将普通函数用做HTTP处理程序。
// 若 f 是个具有适当签名的函数，HandlerFunc(f) 就是个调用 f 的处理程序对象。
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(c, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
	f(w, req)
}
```

`HandlerFunc` 是个具有 `ServeHTTP` 方法的类型， 因此该类型的值就能处理HTTP请求。我们来看看该方法的实现：接收者是一个函数 `f`，而该方法调用 `f`。这看起来很奇怪，但不必大惊小怪， 区别在于接收者变成了一个信道，而方法通过该信道发送消息。

为了将 `ArgServer` 实现成HTTP服务器，首先我们得让它拥有合适的签名。

```go
// 实参服务器。
func ArgServer(w http.ResponseWriter, req *http.Request) {
	fmt.Fprintln(w, os.Args)
}
```

`ArgServer` 和 `HandlerFunc` 现在拥有了相同的签名， 因此我们可将其转换为这种类型以访问它的方法，就像我们将 `Sequence` 转换为 `IntSlice` 以访问 `IntSlice.Sort` 那样。 建立代码非常简单：

```go
http.Handle("/args", http.HandlerFunc(ArgServer))
```

当有人访问 `/args` 页面时，安装到该页面的处理程序就有了值 `ArgServer` 和类型 `HandlerFunc`。 HTTP服务器会以 `ArgServer` 为接收者，调用该类型的 `ServeHTTP` 方法，它会反过来调用 `ArgServer`（通过 `f(c, req)`），接着实参就会被显示出来。

在本节中，我们通过一个结构体，一个整数，一个信道和一个函数，建立了一个HTTP服务器， 这一切都是因为接口只是方法的集和，而几乎任何类型都能定义方法。

# 空白标识符

我们在 [`for-range` 循环](https://go-zh.org/doc/effective_go.html#for)和[映射](https://go-zh.org/doc/effective_go.html#映射)中提过几次空白标识符。 空白标识符可被赋予或声明为任何类型的任何值，而其值会被无害地丢弃。它有点像Unix中的 `/dev/null` 文件：它表示只写的值，在需要变量但不需要实际值的地方用作占位符。 我们在前面已经见过它的用法了。

## 多重赋值中的空白标识符

`for range` 循环中对空表标识符的用法是一种具体情况，更一般的情况即为多重赋值。

若某次赋值需要匹配多个左值，但其中某个变量不会被程序使用， 那么用空白标识符来代替该变量可避免创建无用的变量，并能清楚地表明该值将被丢弃。 例如，当调用某个函数时，它会返回一个值和一个错误，但只有错误很重要， 那么可使用空白标识符来丢弃无关的值。

```go
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
```

你偶尔会看见为忽略错误而丢弃错误值的代码，这是种糟糕的实践。请务必检查错误返回， 它们会提供错误的理由。

```go
// 烂代码！若路径不存在，它就会崩溃。
fi, _ := os.Stat(path)
if fi.IsDir() {
	fmt.Printf("%s is a directory\n", path)
}
```

## 未使用的导入和变量

若导入某个包或声明某个变量而不使用它就会产生错误。未使用的包会让程序膨胀并拖慢编译速度， 而已初始化但未使用的变量不仅会浪费计算能力，还有可能暗藏着更大的Bug。 然而在程序开发过程中，经常会产生未使用的导入和变量。虽然以后会用到它们， 但为了完成编译又不得不删除它们才行，这很让人烦恼。空白标识符就能提供一个工作空间。

这个写了一半的程序有两个未使用的导入（`fmt` 和 `io`）以及一个未使用的变量（`fd`），因此它不能编译， 但若到目前为止代码还是正确的，我们还是很乐意看到它们的。

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
}
```

要让编译器停止关于未使用导入的抱怨，需要空白标识符来引用已导入包中的符号。 同样，将未使用的变量 `fd` 赋予空白标识符也能关闭未使用变量错误。 该程序的以下版本可以编译。

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done. // 用于调试，结束时删除。
var _ io.Reader    // For debugging; delete when done. // 用于调试，结束时删除。

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```

按照惯例，我们应在导入并加以注释后，再使全局声明导入错误静默，这样可以让它们更易找到， 并作为以后清理它的提醒。

## 为副作用而导入

像前例中 `fmt` 或 `io` 这种未使用的导入总应在最后被使用或移除： 空白赋值会将代码标识为工作正在进行中。但有时导入某个包只是为了其副作用， 而没有任何明确的使用。例如，在 `net/http/pprof` 包的 `init` 函数中记录了HTTP处理程序的调试信息。它有个可导出的API， 但大部分客户端只需要该处理程序的记录和通过网页访问数据。只为了其副作用来导入该包， 只需将包重命名为空白标识符：

```go
import _ "net/http/pprof"
```

这种导入格式能明确表示该包是为其副作用而导入的，因为没有其它使用该包的可能： 在此文件中，它没有名字。（若它有名字而我们没有使用，编译器就会拒绝该程序。）

## 接口检查

就像我们在前面[接口](https://go-zh.org/doc/effective_go.html#接口与类型)中讨论的那样， 一个类型无需显式地声明它实现了某个接口。取而代之，该类型只要实现了某个接口的方法， 其实就实现了该接口。在实践中，大部分接口转换都是静态的，因此会在编译时检测。 例如，将一个 `*os.File` 传入一个预期的 `io.Reader` 函数将不会被编译， 除非 `*os.File` 实现了 `io.Reader` 接口。

尽管有些接口检查会在运行时进行。`encoding/json` 包中就有个实例它定义了一个 `Marshaler` 接口。当JSON编码器接收到一个实现了该接口的值，那么该编码器就会调用该值的编组方法， 将其转换为JSON，而非进行标准的类型转换。 编码器在运行时通过[类型断言](https://go-zh.org/doc/effective_go.html#接口转换)检查其属性，就像这样：

```go
m, ok := val.(json.Marshaler)
```

若只需要判断某个类型是否是实现了某个接口，而不需要实际使用接口本身 （可能是错误检查部分），就使用空白标识符来忽略类型断言的值：

```go
if _, ok := val.(json.Marshaler); ok {
	fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```

当需要确保某个包中实现的类型一定满足该接口时，就会遇到这种情况。 若某个类型（例如 `json.RawMessage`） 需要一种定制的JSON表现时，它应当实现 `json.Marshaler`， 不过现在没有静态转换可以让编译器去自动验证它。若该类型通过忽略转换失败来满足该接口， 那么JSON编码器仍可工作，但它却不会使用定制的实现。为确保其实现正确， 可在该包中用空白标识符声明一个全局变量：

```go
var _ json.Marshaler = (*RawMessage)(nil)
```

在此声明中，我们调用了一个 `*RawMessage` 转换并将其赋予了 `Marshaler`，以此来要求 `*RawMessage` 实现 `Marshaler`，这时其属性就会在编译时被检测。 若 `json.Marshaler` 接口被更改，此包将无法通过编译， 而我们则会注意到它需要更新。

在这种结构中出现空白标识符，即表示该声明的存在只是为了类型检查。 不过请不要为满足接口就将它用于任何类型。作为约定， 仅当代码中不存在静态类型转换时才能这种声明，毕竟这是种罕见的情况。



# 内嵌

Go并不提供典型的、类型驱动的子类化概念，但通过将类型**内嵌**到结构体或接口中， 它就能“借鉴”部分实现。

接口内嵌非常简单。我们之前提到过 `io.Reader` 和 `io.Writer` 接口，这里是它们的定义。

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}

type Writer interface {
	Write(p []byte) (n int, err error)
}
```

`io` 包也导出了一些其它接口，以此来阐明对象所需实现的方法。 例如 `io.ReadWriter` 就是个包含 `Read` 和 `Write` 的接口。我们可以通过显式地列出这两个方法来指明 `io.ReadWriter`， 但通过将这两个接口内嵌到新的接口中显然更容易且更具启发性，就像这样：

```go
// ReadWriter 接口结合了 Reader 和 Writer 接口。
type ReadWriter interface {
	Reader
	Writer
}
```

正如它看起来那样：`ReadWriter` 能够做任何 `Reader` **和** `Writer` 可以做到的事情，它是内嵌接口的联合体 （它们必须是不相交的方法集）。**只有接口能被嵌入到接口中。**

同样的基本想法可以应用在结构体中，但其意义更加深远。`bufio` 包中有 `bufio.Reader` 和 `bufio.Writer` 这两个结构体类型， 它们每一个都实现了与 `io` 包中相同意义的接口。此外，`bufio` 还通过结合 `reader/writer` 并将其内嵌到结构体中，实现了带缓冲的 `reader/writer`：它列出了结构体中的类型，但并未给予它们字段名。

```go
// ReadWriter 存储了指向 Reader 和 Writer 的指针。
// 它实现了 io.ReadWriter。
type ReadWriter struct {
	*Reader  // *bufio.Reader
	*Writer  // *bufio.Writer
}
```

内嵌的元素为指向结构体的指针，当然它们在使用前必须被初始化为指向有效结构体的指针。 `ReadWriter` 结构体可通过如下方式定义：

```go
type ReadWriter struct {
	reader *Reader
	writer *Writer
}
```

但为了提升该字段的方法并满足 `io` 接口，我们同样需要提供转发的方法， 就像这样：

```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
	return rw.reader.Read(p)
}
```

而**通过直接内嵌结构体，我们就能避免如此繁琐。 内嵌类型的方法可以直接引用，这意味着 `bufio.ReadWriter` 不仅包括 `bufio.Reader` 和 `bufio.Writer` 的方法，它还同时满足下列三个接口： `io.Reader`、`io.Writer` 以及 `io.ReadWriter`。**

还有种区分内嵌与子类的重要手段。当内嵌一个类型时，该类型的方法会成为外部类型的方法， 但当它们被调用时，该方法的接收者是内部类型，而非外部的。在我们的例子中，当 `bufio.ReadWriter` 的 `Read` 方法被调用时， 它与之前写的转发方法具有同样的效果；接收者是 `ReadWriter` 的 `reader` 字段，而非 `ReadWriter` 本身。

内嵌同样可以提供便利。这个例子展示了一个内嵌字段和一个常规的命名字段。

```go
type Job struct {
	Command string
	*log.Logger
}
```

`Job` 类型现在有了 `Log`、`Logf` 和 `*log.Logger` 的其它方法。我们当然可以为 `Logger` 提供一个字段名，但完全不必这么做。现在，一旦初始化后，我们就能记录 `Job` 了：

```go
job.Log("starting now...")
```

`Logger` 是 `Job` 结构体的常规字段， 因此我们可在 `Job` 的构造函数中，通过一般的方式来初始化它，就像这样：

```go
func NewJob(command string, logger *log.Logger) *Job {
	return &Job{command, logger}
}
```

或通过复合字面：

```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```

若我们需要直接引用内嵌字段，可以忽略包限定名，直接将该字段的类型名作为字段名， 就像我们在 `ReaderWriter` 结构体的 `Read` 方法中做的那样。 若我们需要访问 `Job` 类型的变量 `job` 的 `*log.Logger`， 可以直接写作 `job.Logger`。若我们想精炼 `Logger` 的方法时， 这会非常有用。

```go
func (job *Job) Logf(format string, args ...interface{}) {
	job.Logger.Logf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

内嵌类型会引入命名冲突的问题，但解决规则却很简单。首先，字段或方法 `X` 会隐藏该类型中更深层嵌套的其它项 `X`。若 `log.Logger` 包含一个名为 `Command` 的字段或方法，`Job` 的 `Command` 字段会覆盖它。

其次，若相同的嵌套层级上出现同名冲突，通常会产生一个错误。若 `Job` 结构体中包含名为 `Logger` 的字段或方法，再将 `log.Logger` 内嵌到其中的话就会产生错误。然而，若重名永远不会在该类型定义之外的程序中使用，那就不会出错。 这种限定能够在外部嵌套类型发生修改时提供某种保护。 因此，就算添加的字段与另一个子类型中的字段相冲突，只要这两个相同的字段永远不会被使用就没问题。

# 并发

## 通过通信共享内存

并发编程是个很大的论题。但限于篇幅，这里仅讨论一些Go特有的东西。

在并发编程中，为实现对共享变量的正确访问需要精确的控制，这在多数环境下都很困难。 Go语言另辟蹊径，它将共享的值通过信道传递，实际上，多个独立执行的线程从不会主动共享。 在任意给定的时间点，只有一个Go程能够访问该值。数据竞争从设计上就被杜绝了。 为了提倡这种思考方式，我们将它简化为一句口号：

> 不要通过共享内存来通信，而应通过通信来共享内存。

这种方法意义深远。例如，引用计数通过为整数变量添加互斥锁来很好地实现。 但作为一种高级方法，通过信道来控制访问能够让你写出更简洁，正确的程序。

我们可以从典型的单线程运行在单CPU之上的情形来审视这种模型。它无需提供同步原语。 现在考虑另一种情况，它也无需同步。现在让它们俩进行通信。若将通信过程看做同步着， 那就完全不需要其它同步了。例如，Unix管道就与这种模型完美契合。 尽管Go的并发处理方式来源于Hoare的通信顺序处理（CSP）， 它依然可以看做是类型安全的Unix管道的实现。

## Go程

我们称之为**Go程**是因为现有的术语—线程、协程、进程等等—无法准确传达它的含义。 Go程具有简单的模型：它是与其它Go程并发运行在同一地址空间的函数。它是轻量级的， 所有小号几乎就只有栈空间的分配。而且栈最开始是非常小的，所以它们很廉价， 仅在需要时才会随着堆空间的分配（和释放）而变化。

Go程在多线程操作系统上可实现多路复用，因此若一个线程阻塞，比如说等待I/O， 那么其它的线程就会运行。Go程的设计隐藏了线程创建和管理的诸多复杂性。

在函数或方法前添加 `go` 关键字能够在新的Go程中调用它。当调用完成后， 该Go程也会安静地退出。（效果有点像Unix Shell中的 `&` 符号，它能让命令在后台运行。）

```go
go list.Sort()  // 并发运行 list.Sort，无需等它结束。
```

函数字面在Go程调用中非常有用。

```go
func Announce(message string, delay time.Duration) {
	go func() {
		time.Sleep(delay)
		fmt.Println(message)
	}()  // 注意括号 - 必须调用该函数。
}
```

在Go中，函数字面都是闭包：其实现在保证了函数内引用变量的生命周期与函数的活动时间相同。

这些函数没什么实用性，因为它们没有实现完成时的信号处理。因此，我们需要信道。

## 信道

信道与映射一样，也需要通过 `make` 来分配内存。其结果值充当了对底层数据结构的引用。 若提供了一个可选的整数形参，它就会为该信道设置缓冲区大小。默认值是零，表示不带缓冲的或同步的信道。

```go
ci := make(chan int)            // 整数类型的无缓冲信道
cj := make(chan int, 0)         // 整数类型的无缓冲信道
cs := make(chan *os.File, 100)  // 指向文件指针的带缓冲信道
```

无缓冲信道在通信时会同步交换数据，它能确保（两个Go程的）计算处于确定状态。

信道有很多惯用法，我们从这里开始了解。在上一节中，我们在后台启动了排序操作。 信道使得启动的Go程等待排序完成。

```go
c := make(chan int)  // 分配一个信道
// 在Go程中启动排序。当它完成后，在信道上发送信号。
go func() {
	list.Sort()
	c <- 1  // 发送信号，什么值无所谓。
}()
doSomethingForAWhile()
<-c   // 等待排序结束，丢弃发来的值。
```

接收者在收到数据前会一直阻塞。若信道是不带缓冲的，那么在接收者收到值前， 发送者会一直阻塞；若信道是带缓冲的，则发送者仅在值被复制到缓冲区前阻塞； 若缓冲区已满，发送者会一直等待直到某个接收者取出一个值为止。

带缓冲的信道可被用作信号量，例如限制吞吐量。在此例中，进入的请求会被传递给 `handle`，它从信道中接收值，处理请求后将值发回该信道中，以便让该 “信号量”准备迎接下一次请求。信道缓冲区的容量决定了同时调用 `process` 的数量上限，因此我们在初始化时首先要填充至它的容量上限。

```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
	sem <- 1 // 等待活动队列清空。
	process(r)  // 可能需要很长时间。
	<-sem    // 完成；使下一个请求可以运行。
}

func Serve(queue chan *Request) {
	for {
		req := <-queue
		go handle(req)  // 无需等待 handle 结束。
	}
}
```

由于数据同步发生在信道的接收端（也就是说发送**发生在**>接受**之前**，参见 [Go内存模型](https://go-zh.org/ref/mem)），因此信号必须在信道的接收端获取，而非发送端。

然而，它却有个设计问题：尽管只有 `MaxOutstanding` 个Go程能同时运行，但 `Serve` 还是为每个进入的请求都创建了新的Go程。其结果就是，若请求来得很快， 该程序就会无限地消耗资源。为了弥补这种不足，我们可以通过修改 `Serve` 来限制创建Go程，这是个明显的解决方案，但要当心我们修复后出现的Bug。

```go
func Serve(queue chan *Request) {
	for req := range queue {
		sem <- 1
		go func() {
			process(req) // 这儿有Bug，解释见下。
			<-sem
		}()
	}
}
```

Bug出现在Go的 `for` 循环中，该循环变量在每次迭代时会被重用，因此 `req` 变量会在所有的Go程间共享，这不是我们想要的。我们需要确保 `req` 对于每个Go程来说都是唯一的。有一种方法能够做到，就是将 `req` 的值作为实参传入到该Go程的闭包中：

```go
func Serve(queue chan *Request) {
	for req := range queue {
		sem <- 1
		go func(req *Request) {
			process(req)
			<-sem
		}(req)
	}
}
```

比较前后两个版本，观察该闭包声明和运行中的差别。 另一种解决方案就是以相同的名字创建新的变量，如例中所示：

```go
func Serve(queue chan *Request) {
	for req := range queue {
		req := req // 为该Go程创建 req 的新实例。
		sem <- 1
		go func() {
			process(req)
			<-sem
		}()
	}
}
```

它的写法看起来有点奇怪

```go
req := req
```

但在Go中这样做是合法且惯用的。你用相同的名字获得了该变量的一个新的版本， 以此来局部地刻意屏蔽循环变量，使它对每个Go程保持唯一。

回到编写服务器的一般问题上来。另一种管理资源的好方法就是启动固定数量的 `handle` Go程，一起从请求信道中读取数据。Go程的数量限制了同时调用 `process` 的数量。`Serve` 同样会接收一个通知退出的信道， 在启动所有Go程后，它将阻塞并暂停从信道中接收消息。

```go
func handle(queue chan *Request) {
	for r := range queue {
		process(r)
	}
}

func Serve(clientRequests chan *Request, quit chan bool) {
	// 启动处理程序
	for i := 0; i < MaxOutstanding; i++ {
		go handle(clientRequests)
	}
	<-quit  // 等待通知退出。
}
```

## 信道中的信道

Go最重要的特性就是信道是一等值，它可以被分配并像其它值到处传递。 这种特性通常被用来实现安全、并行的多路分解。

在上一节的例子中，`handle` 是个非常理想化的请求处理程序， 但我们并未定义它所处理的请求类型。若该类型包含一个可用于回复的信道， 那么每一个客户端都能为其回应提供自己的路径。以下为 `Request` 类型的大概定义。

```go
type Request struct {
	args        []int
	f           func([]int) int
	resultChan  chan int
}
```

客户端提供了一个函数及其实参，此外在请求对象中还有个接收应答的信道。

```go
func sum(a []int) (s int) {
	for _, v := range a {
		s += v
	}
	return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// 发送请求
clientRequests <- request
// 等待回应
fmt.Printf("answer: %d\n", <-request.resultChan)
```

On the server side, the handler function is the only thing that changes.

```go
func handle(queue chan *Request) {
	for req := range queue {
		req.resultChan <- req.f(req.args)
	}
}
```

要使其实际可用还有很多工作要做，这些代码仅能实现一个速率有限、并行、非阻塞RPC系统的 框架，而且它并不包含互斥锁。

## 并行化

这些设计的另一个应用是在多CPU核心上实现并行计算。如果计算过程能够被分为几块 可独立执行的过程，它就可以在每块计算结束时向信道发送信号，从而实现并行处理。

让我们看看这个理想化的例子。我们在对一系列向量项进行极耗资源的操作， 而每个项的值计算是完全独立的。

```go
type Vector []float64

// 将此操应用至 v[i], v[i+1] ... 直到 v[n-1]
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
	for ; i < n; i++ {
		v[i] += u.Op(v[i])
	}
	c <- 1    // 发信号表示这一块计算完成。
}
```

我们在循环中启动了独立的处理块，每个CPU将执行一个处理。 它们有可能以乱序的形式完成并结束，但这没有关系； 我们只需在所有Go程开始后接收，并统计信道中的完成信号即可。

```go
const NCPU = 4  // CPU核心数

func (v Vector) DoAll(u Vector) {
	c := make(chan int, NCPU)  // 缓冲区是可选的，但明显用上更好
	for i := 0; i < NCPU; i++ {
		go v.DoSome(i*len(v)/NCPU, (i+1)*len(v)/NCPU, u, c)
	}
	// 排空信道。
	for i := 0; i < NCPU; i++ {
		<-c    // 等待任务完成
	}
	// 一切完成。
}
```

目前Go运行时的实现默认并不会并行执行代码，它只为用户层代码提供单一的处理核心。 任意数量的Go程都可能在系统调用中被阻塞，而在任意时刻默认只有一个会执行用户层代码。 它应当变得更智能，而且它将来肯定会变得更智能。但现在，若你希望CPU并行执行， 就必须告诉运行时你希望同时有多少Go程能执行代码。有两种途径可意识形态，要么 在运行你的工作时将 `GOMAXPROCS` 环境变量设为你要使用的核心数， 要么导入 `runtime` 包并调用 `runtime.GOMAXPROCS(NCPU)`。 `runtime.NumCPU()` 的值可能很有用，它会返回当前机器的逻辑CPU核心数。 当然，随着调度算法和运行时的改进，将来会不再需要这种方法。

注意不要混淆并发和并行的概念：并发是用可独立执行的组件构造程序的方法， 而并行则是为了效率在多CPU上平行地进行计算。尽管Go的并发特性能够让某些问题更易构造成并行计算， 但Go仍然是种并发而非并行的语言，且Go的模型并不适合所有的并行问题。 关于其中区别的讨论，见 [此博文](http://blog.golang.org/2013/01/concurrency-is-not-parallelism.html)。

## 可能泄露的缓冲区

并发编程的工具甚至能很容易地表达非并发的思想。这里有个提取自RPC包的例子。 客户端Go程从某些来源，可能是网络中循环接收数据。为避免分配和释放缓冲区， 它保存了一个空闲链表，使用一个带缓冲信道表示。若信道为空，就会分配新的缓冲区。 一旦消息缓冲区就绪，它将通过 `serverChan` 被发送到服务器。 `serverChan`.

```go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
	for {
		var b *Buffer
		// 若缓冲区可用就用它，不可用就分配个新的。
		select {
		case b = <-freeList:
			// 获取一个，不做别的。
		default:
			// 非空闲，因此分配一个新的。
			b = new(Buffer)
		}
		load(b)              // 从网络中读取下一条消息。
		serverChan <- b   // 发送至服务器。
	}
}
```

服务器从客户端循环接收每个消息，处理它们，并将缓冲区返回给空闲列表。

```go
func server() {
	for {
		b := <-serverChan    // 等待工作。
		process(b)
		// 若缓冲区有空间就重用它。
		select {
		case freeList <- b:
			// 将缓冲区放大空闲列表中，不做别的。
		default:
			// 空闲列表已满，保持就好。
		}
	}
}
```

客户端试图从 `freeList` 中获取缓冲区；若没有缓冲区可用， 它就将分配一个新的。服务器将 `b` 放回空闲列表 `freeList` 中直到列表已满，此时缓冲区将被丢弃，并被垃圾回收器回收。（`select` 语句中的 `default` 子句在没有条件符合时执行，这也就意味着 `selects` 永远不会被阻塞。）依靠带缓冲的信道和垃圾回收器的记录， 我们仅用短短几行代码就构建了一个可能导致缓冲区槽位泄露的空闲列表。

# 错误

库例程通常需要向调用者返回某种类型的错误提示。之前提到过，Go语言的多值返回特性， 使得它在返回常规的值时，还能轻松地返回详细的错误描述。按照约定，错误的类型通常为 `error`，这是一个内建的简单接口。

```go
type error interface {
	Error() string
}
```

库的编写者通过更丰富的底层模型可以轻松实现这个接口，这样不仅能看见错误， 还能提供一些上下文。例如，`os.Open` 可返回一个 `os.PathError`。

```go
// PathError 记录一个错误以及产生该错误的路径和操作。
type PathError struct {
	Op string    // "open"、"unlink" 等等。
	Path string  // 相关联的文件。
	Err error    // 由系统调用返回。
}

func (e *PathError) Error() string {
	return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

`PathError`的 `Error` 会生成如下错误信息：

```go
open /etc/passwx: no such file or directory
```

这种错误包含了出错的文件名、操作和触发的操作系统错误，即便在产生该错误的调用 和输出的错误信息相距甚远时，它也会非常有用，这比苍白的“不存在该文件或目录”更具说明性。

错误字符串应尽可能地指明它们的来源，例如产生该错误的包名前缀。例如在 `image` 包中，由于未知格式导致解码错误的字符串为“image: unknown format”。

若调用者关心错误的完整细节，可使用类型选择或者类型断言来查看特定错误，并抽取其细节。 对于 `PathErrors`，它应该还包含检查内部的 `Err` 字段以进行可能的错误恢复。

```go
for try := 0; try < 2; try++ {
	file, err = os.Create(filename)
	if err == nil {
		return
	}
	if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
		deleteTempFiles()  // 恢复一些空间。
		continue
	}
	return
}
```

这里的第二条 `if` 是另一种[类型断言](https://go-zh.org/doc/effective_go.html#接口转换)。若它失败， `ok` 将为 `false`，而 `e` 则为`nil`. 若它成功，`ok` 将为 `true`，这意味着该错误属于 `*os.PathError` 类型，而 `e` 能够检测关于该错误的更多信息。

## Panic

向调用者报告错误的一般方式就是将 `error` 作为额外的值返回。 标准的 `Read` 方法就是个众所周知的实例，它返回一个字节计数和一个 `error`。但如果错误时不可恢复的呢？有时程序就是不能继续运行。

为此，我们提供了内建的 `panic` 函数，它会产生一个运行时错误并终止程序 （但请继续看下一节）。该函数接受一个任意类型的实参（一般为字符串），并在程序终止时打印。 它还能表明发生了意料之外的事情，比如从无限循环中退出了。

```go
// 用牛顿法计算立方根的一个玩具实现。
func CubeRoot(x float64) float64 {
	z := x/3   // 任意初始值
	for i := 0; i < 1e6; i++ {
		prevz := z
		z -= (z*z*z-x) / (3*z*z)
		if veryClose(z, prevz) {
			return z
		}
	}
	// 一百万次迭代并未收敛，事情出错了。
	panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```

这仅仅是个示例，实际的库函数应避免 `panic`。若问题可以被屏蔽或解决， 最好就是让程序继续运行而不是终止整个程序。一个可能的反例就是初始化： 若某个库真的不能让自己工作，且有足够理由产生Panic，那就由它去吧。

```go
var user = os.Getenv("USER")

func init() {
	if user == "" {
		panic("no value for $USER")
	}
}
```

## Recover

当 `panic` 被调用后（包括不明确的运行时错误，例如切片检索越界或类型断言失败）， 程序将立刻终止当前函数的执行，并开始回溯Go程的栈，运行任何被推迟的函数。 若回溯到达Go程栈的顶端，程序就会终止。不过我们可以用内建的 `recover` 函数来重新或来取回Go程的控制权限并使其恢复正常执行。

调用 `recover` 将停止回溯过程，并返回传入 `panic` 的实参。 由于在回溯时只有被推迟函数中的代码在运行，因此 `recover` 只能在被推迟的函数中才有效。

`recover` 的一个应用就是在服务器中终止失败的Go程而无需杀死其它正在执行的Go程。

```go
func server(workChan <-chan *Work) {
	for work := range workChan {
		go safelyDo(work)
	}
}

func safelyDo(work *Work) {
	defer func() {
		if err := recover(); err != nil {
			log.Println("work failed:", err)
		}
	}()
	do(work)
}
```

在此例中，若 `do(work)` 触发了Panic，其结果就会被记录， 而该Go程会被干净利落地结束，不会干扰到其它Go程。我们无需在推迟的闭包中做任何事情， `recover` 会处理好这一切。

由于直接从被推迟函数中调用 `recover` 时不会返回 `nil`， 因此被推迟的代码能够调用本身使用了 `panic` 和 `recover` 的库函数而不会失败。例如在 `safelyDo` 中，被推迟的函数可能在调用 `recover` 前先调用记录函数，而该记录函数应当不受Panic状态的代码的影响。

通过恰当地使用恢复模式，`do` 函数（及其调用的任何代码）可通过调用 `panic` 来避免更坏的结果。我们可以利用这种思想来简化复杂软件中的错误处理。 让我们看看 `regexp` 包的理想化版本，它会以局部的错误类型调用 `panic` 来报告解析错误。以下是一个 `error` 类型的 `Error` 方法和一个 `Compile` 函数的定义：

```go
// Error 是解析错误的类型，它满足 error 接口。
type Error string
func (e Error) Error() string {
	return string(e)
}

// error 是 *Regexp 的方法，它通过用一个 Error 触发Panic来报告解析错误。
func (regexp *Regexp) error(err string) {
	panic(Error(err))
}

// Compile 返回该正则表达式解析后的表示。
func Compile(str string) (regexp *Regexp, err error) {
	regexp = new(Regexp)
	// doParse will panic if there is a parse error.
	defer func() {
		if e := recover(); e != nil {
			regexp = nil    // 清理返回值。
			err = e.(Error) // 若它不是解析错误，将重新触发Panic。
		}
	}()
	return regexp.doParse(str), nil
}
```

若 `doParse` 触发了Panic，恢复块会将返回值设为 `nil` —被推迟的函数能够修改已命名的返回值。在 `err` 的赋值过程中， 我们将通过断言它是否拥有局部类型 `Error` 来检查它。若它没有， 类型断言将会失败，此时会产生运行时错误，并继续栈的回溯，仿佛一切从未中断过一样。 该检查意味着若发生了一些像索引越界之类的意外，那么即便我们使用了 `panic` 和 `recover` 来处理解析错误，代码仍然会失败。

通过适当的错误处理，`error` 方法（由于它是个绑定到具体类型的方法， 因此即便它与内建的 `error` 类型名字相同也没有关系） 能让报告解析错误变得更容易，而无需手动处理回溯的解析栈：

```go
if pos == 0 {
	re.error("'*' illegal at start of expression")
}
```

尽管这种模式很有用，但它应当仅在包内使用。`Parse` 会将其内部的 `panic` 调用转为 `error` 值，它并不会向调用者暴露出 `panic`。这是个值得遵守的良好规则。

顺便一提，这种重新触发Panic的惯用法会在产生实际错误时改变Panic的值。 然而，不管是原始的还是新的错误都会在崩溃报告中显示，因此问题的根源仍然是可见的。 这种简单的重新触发Panic的模型已经够用了，毕竟他只是一次崩溃。 但若你只想显示原始的值，也可以多写一点代码来过滤掉不需要的问题，然后用原始值再次触发Panic。 这里就将这个练习留给读者了。

# 一个Web服务器

让我们以一个完整的Go程序作为结束吧，一个Web服务器。该程序其实只是个Web服务器的重用。 Google在[http://chart.apis.google.com](http://chart.apis.google.com/) 上提供了一个将表单数据自动转换为图表的服务。不过，该服务很难交互， 因为你需要将数据作为查询放到URL中。此程序为一种数据格式提供了更好的的接口： 给定一小段文本，它将调用图表服务器来生成二维码（QR码），这是一种编码文本的点格矩阵。 该图像可被你的手机摄像头捕获，并解释为一个字符串，比如URL， 这样就免去了你在狭小的手机键盘上键入URL的麻烦。

以下为完整的程序，随后有一段解释。

```go
package main

import (
    "flag"
    "html/template"
    "log"
    "net/http"
)

var addr = flag.String("addr", ":1718", "http service address") // Q=17, R=18

var templ = template.Must(template.New("qr").Parse(templateStr))

func main() {
    flag.Parse()
    http.Handle("/", http.HandlerFunc(QR))
    err := http.ListenAndServe(*addr, nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}

func QR(w http.ResponseWriter, req *http.Request) {
    templ.Execute(w, req.FormValue("s"))
}

const templateStr = `
<html>
<head>
<title>QR Link Generator</title>
</head>
<body>
{{if .}}
<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}" />
<br>
{{.}}
<br>
<br>
{{end}}
<form action="/" name=f method="GET"><input maxLength=1024 size=70
name=s value="" title="Text to QR Encode"><input type=submit
value="Show QR" name=qr>
</form>
</body>
</html>
`
```

`main` 之前的代码应该比较容易理解。我们通过一个标志为服务器设置了默认端口。 模板变量 `templ` 正式有趣的地方。它构建的HTML模版将会被服务器执行并显示在页面中。 稍后我们将详细讨论。

`main` 函数解析了参数标志并使用我们讨论过的机制将 `QR` 函数绑定到服务器的根路径。然后调用 `http.ListenAndServe` 启动服务器；它将在服务器运行时处于阻塞状态。

`QR` 仅接受包含表单数据的请求，并为表单值 `s` 中的数据执行模板。

模板包 `html/template` 非常强大；该程序只是浅尝辄止。 本质上，它通过在运行时将数据项中提取的元素（在这里是表单值）传给 `templ.Execute` 执行因而重写了HTML文本。 在模板文本（`templateStr`）中，双大括号界定的文本表示模板的动作。 从 `{{if .}}` 到 `{{end}}` 的代码段仅在当前数据项（这里是点 `.`）的值非空时才会执行。 也就是说，当字符串为空时，此部分模板段会被忽略。

其中两段 `{{.}}` 表示要将数据显示在模板中 （即将查询字符串显示在Web页面上）。HTML模板包将自动对文本进行转义， 因此文本的显示是安全的。

余下的模板字符串只是页面加载时将要显示的HTML。如果这段解释你无法理解，请参考 [文档](https://go-zh.org/pkg/html/template/) 获得更多有关模板包的解释。

你终于如愿以偿了：以几行代码实现的，包含一些数据驱动的HTML文本的Web服务器。 Go语言强大到能让很多事情以短小精悍的方式解决。

