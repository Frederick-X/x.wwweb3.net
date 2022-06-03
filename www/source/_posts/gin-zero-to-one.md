---
title: Gin Web Framework 从入门到精通
date: 2022-06-03 23:40:41
updated: 2022-06-03 23:40:41
tags: [gin, golang]
---

Gin是用Go（Golang）编写的HTTP Web框架。它具有类似马提尼的API，但性能比马提尼快40倍。如果你追求极致的性能，不妨试试Gin。

<!-- more -->

## Quickstart

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







