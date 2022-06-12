---
title: go语言rpc开发
date: 2022-06-13 00:37:44
updated: 2022-06-13 00:37:44
tags: [go, rpc]
---

go语言内置了`net/rpc`包，本文通过几个案例介绍`net/rpc`包的使用。

<!--more-->

## 快速开始

{% folding open:true Server %}

```go
package main

import (
	"net"
	"net/rpc"
)

type HelloService struct {
}

func (s *HelloService) Hello(request string, reply *string) error {
	// 返回值是通过修改reply的值
	*reply = "Hello, " + request
	return nil
}

func main() {
	listen, err := net.Listen("tcp", ":1234")
	if err != nil {
		return
	}
	err = rpc.RegisterName("HelloService", &HelloService{})
	if err != nil {
		return
	}
	conn, err := listen.Accept()
	rpc.ServeConn(conn)
}
```

{% endfolding %}

{% folding open:true Client %}

```go
package main

import (
	"fmt"
	"net/rpc"
)

func main() {
	client, err := rpc.Dial("tcp", ":1234")
	if err != nil {
		panic("connect err")
	}
	var reply *string = new(string)
	err = client.Call("HelloService.Hello", "world!", reply)
	if err != nil {
		panic("call err")
	}
	fmt.Println(*reply)
}
```

{% endfolding %}

## 关于跨语言调用

go语言rpc的序列化协议是使用go语言特有的Gob编码，这限制了跨语言的调用。而几乎所有的主流语言都支持json，这个例子介绍将go语言的序列化协议改为json，并使用python调用这个rpc服务，实现rpc的跨语言调用。

{% folding open:true Server %}

```go
package main

import (
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
)

type HelloService struct {
}

func (s *HelloService) Hello(request string, reply *string) error {
	*reply = "Hello, " + request
	return nil
}

func main() {
	listen, err := net.Listen("tcp", ":1234")
	if err != nil {
		return
	}
	err = rpc.RegisterName("HelloService", &HelloService{})
	if err != nil {
		return
	}
	for {
		conn, err := listen.Accept()
		if err != nil {
			return
		}
		go rpc.ServeCodec(jsonrpc.NewServerCodec(conn))
	}

}
```

{% endfolding %}

{% folding Go Client %}

```go
package main

import (
	"fmt"
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
)

func main() {
	conn, err := net.Dial("tcp", ":1234")
	if err != nil {
		panic("connect err")
	}
	var reply *string = new(string)
	client := rpc.NewClientWithCodec(jsonrpc.NewClientCodec(conn))
	err = client.Call("HelloService.Hello", "world!", reply)
	if err != nil {
		panic("call err")
	}
	fmt.Println(*reply)
}
```

{% endfolding %}

{% folding open:true Python Client %}

```python
import json
import socket

request = {
    "id": 0,
    "params": ["world!"],
    "method": "HelloService.Hello"
}
client = socket.create_connection(("localhost", 1234))
client.sendall(json.dumps(request).encode())

# 获取服务器返回的数据
rsp = client.recv(4096)
rsp = json.loads(rsp.decode())
print(rsp)
```

`{'id': 0, 'result': 'Hello, world!', 'error': None}`

{% endfolding %}

## 基于http的rpc

{% folding open:true Server %}

```go
package main

import (
	"io"
	"net/http"
	"net/rpc"
	"net/rpc/jsonrpc"
)

type HelloService struct {
}

func (s *HelloService) Hello(request string, reply *string) error {
	*reply = "Hello, " + request
	return nil
}

func main() {
	err := rpc.RegisterName("HelloService", &HelloService{})
	if err != nil {
		return
	}
	http.HandleFunc("/jsonrpc", func(w http.ResponseWriter, r *http.Request) {
		var conn io.ReadWriteCloser = struct {
			io.Writer
			io.ReadCloser
		}{
			ReadCloser: r.Body,
			Writer:     w,
		}
		rpc.ServeCodec(jsonrpc.NewServerCodec(conn))
	})
	http.ListenAndServe(":1234", nil)
}
```

{% endfolding %}

{% folding open:true Python Client %}

```python
import requests
request = {
    "id": 0,
    "params": ["world!"],
    "method": "HelloService.Hello"
}
rsp = requests.post("http://localhost:1234/jsonrpc", json=request)

print(rsp.text)
```

`{"id":0,"result":"Hello, world!","error":null}`

{% endfolding %}

## 改进rpc调用

首先我们回顾一下上面的例子。在上面的例子中，客户端通过`client.Call("HelloService.Hello", "world!", reply)` 实现函数的调用。我们不仅需要记住函数的名字`HelloService.Hello` , 而且函数参数的传递方式也与普通函数的调用有一定的差异。

这一节就以上问题做进一步优化。

项目目录结构如下：

```
.
├── GoProject
│   ├── go.mod
│   ├── go.sum
│   ├── helloworld_rpc
│   │   ├── client
│   │   │   └── client.go
│   │   ├── client_proxy
│   │   │   └── client_proxy.go
│   │   ├── handler
│   │   │   └── hanlder.go
│   │   ├── server
│   │   │   └── server.go
│   │   └── server_proxy
│   │       └── server_proxy.go
```

{% folding open:true hanlder.go %}

```go
package handler

const HelloServiceName = "handler/HelloService"

type HelloService struct {
}

func (s *HelloService) Hello(request string, reply *string) error {
	*reply = "Hello, " + request
	return nil
}

```

{% endfolding %}

{% folding open:true client_proxy.go %}

```go
package client_proxy

import (
	"GoProject/helloworld_rpc/handler"
	"net/rpc"
)

type HelloServiceStub struct {
	*rpc.Client
}

func NewHelloServiceClient(protocol, address string) HelloServiceStub {
	conn, err := rpc.Dial(protocol, address)
	if err != nil {
		panic("connect err")
	}
	return HelloServiceStub{conn}
}

func (c *HelloServiceStub) Hello(request string, reply *string) error {
	err := c.Call(handler.HelloServiceName+".Hello", request, reply)
	if err != nil {
		return err
	}
	return nil
}
```

{% endfolding %}

{% folding open:true server_proxy.go %}

```go
package server_proxy

import (
	"GoProject/helloworld_rpc/handler"
	"net/rpc"
)

type HelloServicer interface {
	Hello(request string, reply *string) error
}

func RegisterHelloService(srv HelloServicer) error {
	return rpc.RegisterName(handler.HelloServiceName, srv)
}
```

{% endfolding %}

{% folding open:true server.go %}

```go
package main

import (
	"GoProject/helloworld_rpc/handler"
	"GoProject/helloworld_rpc/server_proxy"
	"net"
	"net/rpc"
)

func main() {
	listen, err := net.Listen("tcp", ":1234")
	if err != nil {
		return
	}
	server_proxy.RegisterHelloService(&handler.HelloService{})
	if err != nil {
		return
	}
	conn, err := listen.Accept()
	rpc.ServeConn(conn)
}
```



{% endfolding %}

{% folding open:true client.go %}

```go
package main

import (
	"GoProject/helloworld_rpc/client_proxy"
	"fmt"
)

func main() {
	client := client_proxy.NewHelloServiceClient("tcp", ":1234")

	var reply *string = new(string)
	err := client.Hello("world!", reply)
	if err != nil {
		panic("call err")
	}
	fmt.Println(*reply)
}
```

{% endfolding %}



