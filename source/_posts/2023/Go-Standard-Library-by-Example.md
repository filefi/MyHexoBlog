---
title: Go Standard Library by Example
date: 2023-05-28 18:25:04
updated: 2023-05-28 18:25:04
tags: [Golang]
categories: Golang
---

<!-- more -->

# `io` 基本I/O接口
`io` 包为 I/O 原语提供了基本的接口。它主要包装了这些原语的已有实现。

由于这些被接口包装的I/O原语是由不同的低级操作实现，因此，在另有声明之前不该假定它们的并发执行是安全的。

在 `io` 包中最重要的是两个接口：`Reader` 和 `Writer` 接口。

## `Reader` 接口
```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```
`Reader`接口封装了基本的 `Read` 方法。

`Read` 将 `len(p)` 个字节读取到 `p` 中。它返回读取的字节数 `n（0 <= n <= len(p)）` 以及任何遇到的错误。即使 `Read` 返回的 `n < len(p)`，它也会在调用过程中占用 `len(p)` 个字节作为暂存空间。若可读取的数据不到 `len(p)` 个字节，`Read` 会返回可用数据，而不是等待更多数据。

当 `Read` 在成功读取 `n > 0` 个字节后遇到一个错误或 EOF (end-of-file)，它会返回读取的字节数。它可能会同时在本次的调用中返回一个non-nil错误,或在下一次的调用中返回这个错误（且 n 为 0）。 一般情况下, Reader会返回一个非0字节数n, 若 `n = len(p)` 个字节从输入源的结尾处由 Read 返回，`Read` 可能返回 `err == EOF` 或者 `err == nil`。并且之后的 `Read()` 都应该返回 (n:0, err:EOF)。

调用者在考虑错误之前应当首先处理返回的数据。这样做可以正确地处理在读取一些字节后产生的 I/O 错误，同时允许EOF的出现。

`ReadFrom` 函数将 `io.Reader` 作为参数，也就是说，`ReadFrom` 可以从任意的地方读取数据，只要来源实现了 `io.Reader` 接口。比如，我们可以从标准输入、文件、字符串等读取数据，示例代码如下：
```go
func ReadFrom(reader io.Reader, num int) ([]byte, error) {
    p := make([]byte, num)
    n, err := reader.Read(p)
    if n > 0 {
        return p[:n], nil
    }
    return p, err
}

// 从标准输入读取
data, err = ReadFrom(os.Stdin, 11)

// 从普通文件读取，其中 file 是 os.File 的实例
data, err = ReadFrom(file, 9)

// 从字符串读取
data, err = ReadFrom(strings.NewReader("from string"), 12)
```


### func LimitReader(r Reader, n int64) Reader

### func MultiReader(readers ...Reader) Reader

### func TeeReader(r Reader, w Writer) Reader


## `ByteReader` 接口

## type ByteScanner

## type ByteWriter

## type Closer

## type LimitedReader
### func (l *LimitedReader) Read(p []byte) (n int, err error)

## type OffsetWriter
### func NewOffsetWriter(w WriterAt, off int64) *OffsetWriter
### func (o *OffsetWriter) Seek(offset int64, whence int) (int64, error)
### func (o *OffsetWriter) Write(p []byte) (n int, err error)
### func (o *OffsetWriter) WriteAt(p []byte, off int64) (n int, err error)

## type PipeReader
### func (r *PipeReader) Close() error
### func (r *PipeReader) CloseWithError(err error) error
### func (r *PipeReader) Read(data []byte) (n int, err error)

## type PipeWriter
### func (w *PipeWriter) Close() error
### func (w *PipeWriter) CloseWithError(err error) error
### func (w *PipeWriter) Write(data []byte) (n int, err error)

## type ReadCloser
### func NopCloser(r Reader) ReadCloser

## type ReadSeekCloser

## type ReadSeeker

## type ReadWriteCloser

## type ReadWriteSeeker

## type ReadWriter

## type Reader


## type ReaderAt

## type ReaderFrom

## type RuneReader

## type RuneScanner

## type SectionReader
### func NewSectionReader(r ReaderAt, off int64, n int64) *SectionReader
### func (s *SectionReader) Read(p []byte) (n int, err error)
### func (s *SectionReader) ReadAt(p []byte, off int64) (n int, err error)
### func (s *SectionReader) Seek(offset int64, whence int) (int64, error)
### func (s *SectionReader) Size() int64

## type Seeker

## type StringWriter

## type WriteCloser

## type WriteSeeker

## type Writer
### func MultiWriter(writers ...Writer) Writer

## type WriterAt

## type WriterTo