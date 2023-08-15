---
title: 使用 colly.Context 在 colly.Collector 之间传递自定义数据
date: 2023-08-15 21:54:03
updated: 2023-08-15 21:54:03
tags: [Golang, Colly, Web Scraping, Spider, Scraper, Crawler]
categories: [Golang]
---


爬虫通常需要进行**横向爬取**和**纵向爬取**：

- **横向**：从一个索引页到另外一个索引页。也叫做**水平爬取**，因为这种情况是在同一层级下爬取页面（比如索引页或列表页）。

- **纵向**：从一个索引页到详情页，并在详情页中抽取数据填充Item。也叫做**垂直爬取**，因为这种方式是从一个更高的层级（比如索引页）到一个更低的层级（比如详情页）。

Colly官方文档建议，如果单个爬取作业过于复杂，或者有多个不同的子任务，则应该使用多个`colly.Colletor`负责不同的子任务。比如，一个负责横向，从索引页获得更多链接。一个负责纵向，爬取详情页中的数据或下载文件。

编写爬虫时，经常需要把一个页面中爬取到的数据传递给下一个请求。例如，由页面A中的数据拼接出下一个Request的URL；或者，由页面A中的数据拼接得到文件名，并将文件名传递给下载文件的Request，用来设置下载文件的文件名。

在 Scrapy 中，我们可以通过`Request`的`meta`属性向`Response`传递数据。例如：

```python
def start_requests(self):
    # 将数据存放如 request的meta属性中
  	yield Request(url, meta={'name':'Mike', 'age':12})

def parse(self, response):
		item = Item()
    # 从response.meta中取出数据
    item['name'] = response.meta['name']
    item['age'] = response.meta['age']
```

或者

```python
def start_requests(self):
  	request = Request(url)
    # 将数据存放如 request的meta属性中
    request.meta = {'name':'Mike', 'age':12}
    yield request

def parse(self, response):
		item = Item()
    # 从response.meta中取出数据
    item['name'] = response.meta['name']
    item['age'] = response.meta['age']
```

那么，在 [Colly](http://go-colly.org/) 中该如何在 `colly.Collector` 之间传递数据呢？



<!-- more -->



借助 [`colly.Context`](https://github.com/gocolly/colly/blob/v2.1.0/context.go#L22) ，我们可以在 `colly.Collector` 之间传递自定义数据。

```go
type Context struct {
	// contains filtered or unexported fields
}
```

根据官方文档，[`colly.Context`](https://github.com/gocolly/colly/blob/v2.1.0/context.go#L22) 用来在 callbacks 之间传递数据。

[`colly.Context`](https://github.com/gocolly/colly/blob/v2.1.0/context.go#L22)相关的函数：

- [`NewContext`](https://pkg.go.dev/github.com/gocolly/colly/v2#NewContext): `NewContext` initializes a new `Context` instance.

[`colly.Context`](https://github.com/gocolly/colly/blob/v2.1.0/context.go#L22)的方法：

-  [`Put`](https://pkg.go.dev/github.com/gocolly/colly/v2#Context.Put): `Put` stores a value of any type in `Context`.
- [`Get`](https://pkg.go.dev/github.com/gocolly/colly/v2#Context.Get): `Get` retrieves a string value from `Context`. `Get` returns an empty string if key not found.
- [`GetAny`](https://pkg.go.dev/github.com/gocolly/colly/v2#Context.GetAny): `GetAny` retrieves a value from `Context`. `GetAny` returns nil if key not found.
- [`ForEach`](https://pkg.go.dev/github.com/gocolly/colly/v2#Context.ForEach): `ForEach` iterate context.
- [`MarshalBinary`](https://pkg.go.dev/github.com/gocolly/colly/v2#Context.MarshalBinary): `MarshalBinary` encodes `Context` value This function is used by request caching.
- [`UnmarshalBinary`](https://pkg.go.dev/github.com/gocolly/colly/v2#Context.UnmarshalBinary): `UnmarshalBinary` decodes `Context` value to nil This function is used by request caching.

实例：

```go
package main

import (
	"fmt"

	"github.com/gocolly/colly/v2"
)

func main() {
	c := colly.NewCollector(
		colly.UserAgent("myUserAgent"),
		colly.AllowedDomains("example.com"),
	)
	c2 := c.Clone()

	c.OnResponse(func(r *colly.Response) {
		// data items you scraped
		filename := "custom_name.html"
		url := "http://example.com/index.html"

		// Context provides a tiny layer for passing data between callbacks
		// Put stores a value of any type in Context
		r.Ctx.Put("filename", filename)
		r.Ctx.Put("url", url)

		// Use collector’s Request() function to be able to share context with other collectors.
		c2.Request("GET", url, nil, r.Ctx, nil)
	})

	c2.OnResponse(func(r *colly.Response) {
		// Get retrieves a string value from Context. Get returns an empty string if key not found
		filename := r.Ctx.Get("filename")
		url := r.Ctx.Get("url")
		fmt.Printf("filename:%s url:%s", filename, url)

		// now you can custom the path
		r.Save(filename)
	})

	c.Visit("http://example.com")

	// r.Ctx.Put("item", struct {
	// 	Name string
	// 	Age  int
	// }{Name: "neo", Age: 12})

}
```
