---
title: go + python 快速入门grpc
date: 2022-06-13 22:42:10
updated: 2022-06-13 22:42:10
categories: [技术加油站]
tags: [go, python, grpc, 微服务]
---

gRPC是一个由Google开发的高性能、开源、通用的远程过程调用（RPC）框架。其面向移动应用开发并给予HTTP/2协议标准以及ProtoBuf(Protocol Buffers)序列化协议设计开发。

<!--more-->

gRPC可以高效地连接数据中心内和之间的服务，并可插拔地支持负载平衡、跟踪、运行状况检查和身份验证。它还适用于最后一英里的分布式计算，将设备、移动应用程序和浏览器连接到后端服务。目前提供C、JAVA和Go语言版本，分别是grpc、grpc-java、grpc-go。

{% tabs active:3 align:center %}

<!-- tab grpc -->

{% ghcard grpc/grpc %}

<!-- tab grpc-java -->

{% ghcard grpc/grpc-java %}

<!-- tab grpc-go -->

{% ghcard grpc/grpc-go %}

{% endtabs %}



## gRPC与Protocol Buffers简介

本小节介绍gRPC和protocol buffers。gRPC可以使用protocol buffers作为其接口定义语言（Interface Definition Language, IDL）和底层消息交换格式。

### gRPC

在gRPC中，客户端程序可以直接像调用本地方法一般调用另一台机器上的服务器程序中的方法。与常见的RPC系统一样，gRPC通过定义服务，定义可以使用参数和返回类型远程调用的方法。在服务器端，服务器实现此接口并运行gRPC服务器来处理客户端调用。在客户端，客户端有一个**stub**，提供与服务器相同的方法。

{% image landing-2.svg gRPC数据交互示意图 %}

### Protocol Buffers

gRPC默认使用Protocol Buffers。Protocol Buffers是由Google开源的序列化结构化数据的成熟机制，其压缩比高，传输速度快，序列化/反序列化性能较Json、XML等快2-100倍。除性能优势外，Ptotocol Buffers还有易于使用（自动生成序列化/反序列化代码）、维护成本低（只需维护proto文件）、向后兼容（无需破坏旧格式）、加密性好、跨平台（支持多种主流语言）等优势。

Protocol Buffers的缺点主要体现在其通用型较差，相比于Json，Ptotocol Buffers需要专门的解析工具，此外，Protocol Buffers自解释性差，只有通过proto文件才能了解其数据结构。

## python中使用protocol buffers

安装依赖库

```
pip install grpcio
pip install grpcio-tools
```

Protocol Buffers（以下简称ProtoBuf） 有ProtoBuf2和ProtoBuf3两个版本，gRPC官网更加推荐使用ProtoBuf3:

> ... In general, while you can use proto2 (the current default protocol buffers version), we recommend that you use proto3 with gRPC as it lets you use the full range of gRPC-supported languages, as well as avoiding compatibility issues with proto2 clients talking to proto3 servers and vice versa.

ProtoBuf有专门的定义格式。使用ProtoBuf需要在扩展名为`.proto`的文本文件中定义期望序列化的数据结构。举一个例子，文件名为`hello.proto`:

```protobuf
syntax = "proto3";

message HelloRequest {
  string name = 1; // name 表示名称， name的编号是1
}
```

然后，可用如下命令在当前目录生成`hello.proto`对应的python文件。

```bash
python -m grpc_tools.protoc --python_out=. --grpc_python_out=. -I. hello.proto
```

执行上面的命令后，会生成`hello_pb2.py`和`hello_pb2_grpc.py`两个文件。

至此，当前项目目录的结构如下：

```
.
├── main.py
└── proto
    ├── __init__.py
    ├── hello.proto
    ├── hello_pb2.py
    └── hello_pb2_grpc.py
```

> 注：此时`hello.proto`已经完成了它的使命，理论上可以被删除。

在`main.py`中编写如下代码：

```python
from proto import hello_pb2

request = hello_pb2.HelloRequest()
request.name = "World"

res_str = request.SerializeToString()
print(res_str)

request2 = hello_pb2.HelloRequest()
request2.ParseFromString(res_str)
print(request2.name)
```

运行结果：

```
b'\n\x05World'
World
```



## python grpc开发

这里以gRPC GitHub仓库的helloworld example做示例。

`helloworld.proto`文件内容如下：

```protobuf
syntax = "proto3";

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

生成对应的python文件

```
python -m grpc_tools.protoc --python_out=. --grpc_python_out=. -I. hello.proto
```



编写`server.py`，内容如下：

```python
from proto import hello_pb2, hello_pb2_grpc
import grpc
from concurrent import futures


class Greeter(hello_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        return hello_pb2.HelloReply(message=f"Hello, {request.name}")


if __name__ == "__main__":
    # Instantiation a Server
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    # Register the logic into the server
    hello_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    # Start server
    server.add_insecure_port('0.0.0.0:50051')
    server.start()
    server.wait_for_termination()
```



编写`client.py`：

```python
from proto import hello_pb2, hello_pb2_grpc
import grpc

if __name__ == '__main__':
    with grpc.insecure_channel("127.0.0.1:50051") as channel:
        stub = hello_pb2_grpc.GreeterStub(channel)
        rsp: hello_pb2.HelloReply = stub.SayHello(hello_pb2.HelloRequest(name='World'))
        print(rsp.message)
```



## go gRPC开发

安装 protobuf编译工具

```
brew install protobuf
```

安装依赖

```
go install github.com/golang/protobuf/protoc-gen-go@v1.4.0
```

创建`hello.proto`文件，除`option go_package = ".;proto";`外，其余部分与python中一致。

```protobuf
syntax = "proto3";
option go_package = ".;proto";
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```



在当前目录下执行

```
protoc -I . hello.proto --go_out=plugins=grpc:.
```

可以看到目录下生成了一个`hello.pb.go`文件



下面开始编写server和client的代码。创建server和client目录，以及源码项目目录结构如下：

```
.
├── GoProject
│   ├── go.mod
│   ├── go.sum
│   ├── grpc_test
│   │   ├── client
│   │   │   └── client.go
│   │   ├── proto
│   │   │   ├── hello.pb.go
│   │   │   └── hello.proto
│   │   └── server
│   │       └── server.go
```



`server.go`代码如下：

```go
package main

import (
	"GoProject/grpc_test/proto"
	"context"
	"google.golang.org/grpc"
	"net"
)

type Server struct{}

func (s *Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply, error) {
	return &proto.HelloReply{
		Message: "hello" + request.Name,
	}, nil
}
func main() {
	g := grpc.NewServer()
	proto.RegisterGreeterServer(g, &Server{})
	lis, err := net.Listen("tcp", "0.0.0.0:50051")
	if err != nil {
		panic("failed to listen:" + err.Error())
	}
	err = g.Serve(lis)
	if err != nil {
		panic("failed to start grpc:" + err.Error())
	}
}
```



`client.go`代码如下。

```go
package main

import (
	"GoProject/grpc_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
)

func main() {
	conn, err := grpc.Dial("127.0.0.1:50051", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	c := proto.NewGreeterClient(conn)
	r, err := c.SayHello(context.Background(), &proto.HelloRequest{Name: "World"})
	if err != nil {
		panic(err)
	}
	fmt.Println(r.Message)
}
```



> 只要proto文件一致，即可实用python的client调用go的server（反之亦然）





## gRPC中的数据流

gRPC中有四种数据流，分别是`简单模式（Simple RPC）` `服务端数据流模式（Server-side streaming RPC）` `客户端数据流模式（Client-side streaming RPC）` `双向数据流模式（Bidirectional streaming RPC）`

`服务端数据流模式`是客户端发起一次请求，服务端返回一段连续的数据流。如Binance App向服务端发起ETH的查询请求，服务端就把ETH的实时数据远远不断返回给客户端。

`客户端数据流模式`，如物联网终端向服务器报送数据。

`双向数据流模式`，如聊天机器人。



### 示例

项目目录如下。

```
.
├── GoProject
│   ├── go.mod
│   ├── go.sum
│   └── stream_grpc_test
│       ├── client
│       │   └── client.go
│       ├── proto
│       │   └── stream.proto
│       └── server
│           └── server.go
```



`stream.proto`

```protobuf
syntax = "proto3";

option go_package=".;proto";

service Greeter {
  rpc GetStream(StreamReqData) returns (stream StreamResData); // 服务端流模式
  rpc PutStream(stream StreamReqData) returns (StreamResData); // 客户端流模式
  rpc AllStream(stream StreamReqData) returns (stream StreamResData); // 双向流模式
}

message StreamReqData {
  string data = 1;
}

message StreamResData {
  string data = 1;
}
```



生成proto文件：

```
protoc -I . stream.proto --go_out=plugins=grpc:.
```



{% folding 🌰 服务端流模式 %}

`server.go`

```go
package main

import (
	"GinTest/stream_grpc_test/proto"
	"fmt"
	"google.golang.org/grpc"
	"net"
	"time"
)

const PORT = ":50052"

type Server struct{}

func (s *Server) GetStream(req *proto.StreamReqData, res proto.Greeter_GetStreamServer) error {
	i := 0
	for {
		i++
		_ = res.Send(&proto.StreamResData{Data: fmt.Sprintf("%v", time.Now().Unix())})
		time.Sleep(time.Second)
		if i >= 10 {
			break
		}
	}

	return nil
}
func (s *Server) PutStream(cliStr proto.Greeter_PutStreamServer) error {
	return nil
}
func (s *Server) AllStream(allStr proto.Greeter_AllStreamServer) error {
	return nil
}

func main() {
	lis, err := net.Listen("tcp", PORT)
	if err != nil {
		panic(err)
	}
	s := grpc.NewServer()
	proto.RegisterGreeterServer(s, &Server{})
	err = s.Serve(lis)
	if err != nil {
		panic("failed to start grpc:" + err.Error())
	}
}
```

`client.go`

```go
package main

import (
	"GinTest/stream_grpc_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
)

func main() {
	conn, err := grpc.Dial("localhost:50052", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	c := proto.NewGreeterClient(conn)
	res, _ := c.GetStream(context.Background(), &proto.StreamReqData{
		Data: "ETH",
	})
	for {
		a, err := res.Recv()
		if err != nil {
			panic(err)
		}
		fmt.Println(a)
	}
}
```



{% endfolding %}



{% folding 🌰客户端流模式 %}

`server.go`

```go
func (s *Server) PutStream(cliStr proto.Greeter_PutStreamServer) error {
	for {
		if a, err := cliStr.Recv(); err != nil {
			panic(err)
		} else {
			fmt.Println(a.Data)
		}
	}
	return nil
}
```

`client.go`

```go
package main

import (
	"GinTest/stream_grpc_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"time"
)

func main() {
	conn, err := grpc.Dial("localhost:50052", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	c := proto.NewGreeterClient(conn)
	
	putS, _ := c.PutStream(context.Background())
	i := 0
	for {
		i++
		err := putS.Send(&proto.StreamReqData{
			Data: fmt.Sprintf("%d", i),
		})
		if err != nil {
			return
		}
		time.Sleep(time.Second)
		if i >= 10 {
			break
		}
	}
}
```

{% endfolding %}



{% folding 🌰双向流模式 %}

`server.go`

```go
func (s *Server) AllStream(allStr proto.Greeter_AllStreamServer) error {
	wg := sync.WaitGroup{}
	wg.Add(2)
	go func() {
		defer wg.Done()
		for {
			data, _ := allStr.Recv()
			fmt.Println("收到客户端消息：" + data.Data)
		}
	}()

	go func() {
		defer wg.Done()
		for {
			_ = allStr.Send(&proto.StreamResData{Data: "我是服务器"})
			time.Sleep(time.Second)
		}
	}()
	wg.Wait()
	return nil
}
```

`client.go`

```go
allS, _ := c.AllStream(context.Background())
	wg := sync.WaitGroup{}
	wg.Add(2)
	go func() {
		defer wg.Done()
		for {
			data, _ := allS.Recv()
			fmt.Println("收到客户端消息：" + data.Data)
		}
	}()

	go func() {
		defer wg.Done()
		for {
			_ = allS.Send(&proto.StreamReqData{Data: "我是客户端"})
			time.Sleep(time.Second)
		}
	}()
	wg.Wait()
```

{% endfolding %}



