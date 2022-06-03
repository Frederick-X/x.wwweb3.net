---
title: Gin Web Framework 从入门到精通
date: 2022-06-03 23:40:41
updated: 2022-06-03 23:40:41
tags: [gin, golang]
---

Gin是用Go（Golang）编写的HTTP Web框架。它具有类似Martini的API，但性能比Martini快40倍。如果你追求极致的性能，不妨试试Gin。

<!-- more -->

{% ghcard gin-gonic/gin theme:dark %}

## 快速开始

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})
	r.Run(":8080")
}
```

{% folding gin.Default()和gin.New()的区别 %}

```go
r := gin.Default()
r := gin.New()
```

二者之处在于使用`gin.Default()`会自动开启`logger()`和`Recovery()`。
{% endfolding %}

{% folding 配置GET，POST，PATCH等HTTP方法 %}

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	r.GET("/sthGet", getting)
	r.POST("/sthPost", posting)
	r.PUT("/sthPut", putting)
	r.DELETE("/sthDelete", deleting)
	r.PATCH("/sthPatch", patching)
	r.HEAD("/sthHead", head)
	r.OPTIONS("sthOptions", options)
	r.Run(":8080")
}
```

{% endfolding %}

{% folding 路由分组 %}

随着业务的复杂，API数量也随之增多，通常需要对路由进行分组（如接口的版本管理）。

```go
r := gin.Default()
v1 := r.Group("/v1")
{
	v1.POST("/login", loginFunc)
	v1.POST("/submit", submitFunc)
}
r.Run(":8080")
```

{% endfolding %}

{% folding 匹配url中的变量 %}

```go
func main() {
	r := gin.Default()
	book := r.Group("/book")
	{
		book.GET("/:id", bookDetail)
	}
	r.Run(":8080")
}

func bookDetail(c *gin.Context) {
	id := c.Param("id")
	c.JSON(http.StatusOK, gin.H{
		"id": id,
	})
}
```

{% endfolding %}

{% folding 匹配url中的路径 %}

```go
func main() {
	r := gin.Default()
	book := r.Group("/book")
	{
		book.GET("/:id/*file", bookDetail)
	}
	r.Run(":8080")
}

func bookDetail(c *gin.Context) {
	id := c.Param("id")
	file := c.Param("file")
	c.JSON(http.StatusOK, gin.H{
		"id":   id,
		"file": file,
	})
}
```

```
url: 
	http://127.0.0.1:8080/book/2/usr/books/mybook.txt
output:
  {
    "file": "/usr/books/mybook.txt",
    "id": "2"
  }
```

{% endfolding %}

## 入门

### ShouldBindUri实现uri的模式约束

在[快速开始](#快速开始)末尾处的例子介绍了如何获取url中的变量。而这些例子都无法约束参数的类型。

下面的例子介绍使用ShouldBindUri方法提取uri中的参数。

```go
type Book struct {
	ID   int    `uri:"id" binding:"required"`
	Name string `uri:"name" binding:"required"`
}

func main() {
	r := gin.Default()
	book := r.Group("/book")
	{
		book.GET("/:id/:name", bookDetail)
	}
	r.Run(":8080")
}

func bookDetail(c *gin.Context) {
	var book Book
	if err := c.ShouldBindUri(&book); err != nil {
		c.Status(404)
		return
	}
	c.JSON(http.StatusOK, gin.H{
		"id":   book.ID,
		"name": book.Name,
	})
}
```

### 从GET和POST中获取参数

{% folding 获取GET参数 %}

```go
func main() {
	r := gin.Default()
	book := r.Group("/book")
	{
		//url: http://127.0.0.1:8080/book?id=2&name=mybook
		book.GET("", bookDetail)
	}
	r.Run(":8080")
}

func bookDetail(c *gin.Context) {
	bookId := c.Query("id")
	bookName := c.DefaultQuery("name", "None") // defaultValue is needed
	c.JSON(http.StatusOK, gin.H{
		"id":   bookId,
		"name": bookName,
	})
}
```

{% endfolding %}

{% folding 获取POST参数 %}

```go
func main() {
	r := gin.Default()
	book := r.Group("/book")
	{
		book.POST("", createBook)
	}
	r.Run(":8080")
}

func createBook(c *gin.Context) {
	bookId := c.PostForm("id")
	bookName := c.DefaultPostForm("name", "None") // defaultValue is needed
	c.JSON(http.StatusOK, gin.H{
		"id":   bookId,
		"name": bookName,
	})
}
```

{% endfolding %}
