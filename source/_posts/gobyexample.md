---
title: Go by Example
date: 2021-12-07 11:11:57
tags:Golang
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

