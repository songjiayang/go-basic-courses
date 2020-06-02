# TCP

从本节开始，我将向大家讲解有关 Go 网络的知识，主要分为传输层协议 TCP 和 UDP，应用层协议 HTTP 和 WebSocket，而本节将着重介绍 TCP。

TCP 名叫传输控制协议（英语：Transmission Control Protocol，缩写：TCP）是一种面向连接的、可靠的、基于字节流的传输层通信协议。TCP 是位于 IP 层和应用层之间的传输层，它是目前互联网通信的重要基石，是主要应用层协议 HTTP、WebSocket 等的底层支持。

参见 TCP/IP 4 层的分层图：

![4layer.png](4layer.png)

Go 语言中关于网络的操作都封装在 [net](https://golang.org/pkg/net/) 包中，这个包非常强大，包含了 IP、TCP、UDP 等一系列操作，下面我就带着大家一起来学习 Net 包中的 TCP 的相关用法。

### 基本知识

因为 TCP 是面向连接的传输层协议，所以使用的时候主要分为如下几步：

- 使用 `Listen` 创建一个 TCP Server，并通过 `l.Accept` 获取客户端的连接请求。
- 在 Client 端使用 `Dial` 函数向指定的 TCP 地址（IP:Port) 发起连接请求 (该 Server 可以是自己搭建)。
- 创建成功后，会生成用于通信的连接 `*net.TCPConn`。
- 该 `TCPConn` 是双向的，Client 和 Server 之间可以批次通信。
- 可以通过 `SetReadDeadline` 和 `SetWriteDeadline` 设置读写超时时间。

核心 API 如下：

- 创建 TCP Server:

```
l, err := net.Listen("tcp", "127.0.0.1:3000")
```

>>  `Listen` 其实是 net 包的一个辅助方法，它等价于以下操作：

```
// 指定本地地址
laddr, _ := net.ResolveTCPAddr("tcp", "0.0.0.0:0")
// 指定监听的协议和本地地址
l, err := net.ListenTCP("tcp", laddr)
```

- 等待连接:

```
c, err := l.Accept()
```

>> 当客户端调用 Dail 方法的时候， Server 端可以通过此方法获得和客户端一样的连接，它们分别是一个 Socket 的两端。

- 客户端发起连接请求并创建连接:

```go
c, err := net.Dial("tcp", "ip:port")
```

>>  `Dial` 其实是 net 包的一个辅助方法，它不仅支持 `tcp` 的连接创建，还支持 `udp`, `ip`, `unix`，它等价于以下操作：

```go
// 指定本地地址
laddr, _ := net.ResolveTCPAddr("tcp", "0.0.0.0:0")
// 服务端地址
raddr, _ := net.ResolveTCPAddr("tcp", "ip:port")
// 创建连接
c, err := net.DialTCP("tcp", laddr, raddr)
```

- 通过连接发送数据:

```go
c.Write([]byte("ping"))
```

- 通过连接读取数据：

```go
buf := make([]byte, 1024)
c.Read(buf)
```

- 设置读超时时间:

```go
c.SetReadDeadline(time.Now().Add(defaultTimeout))
```

- 设置写超时时间:

```go
c.SetWriteDeadline(time.Now().Add(defaultTimeout))
```

>> 注意： `SetReadDeadline`、`SetWriteDeadline` 方法的参数为具体的时刻，而不是时间间隔（time.Duration）。

- 关闭连接：

```go
c.Close()
```

>> 注意： 连接用完之后，需要通过此方法来进行资源释放，不然会出现句柄泄露的问题。


### 实战示例

下面我们就来看一个简单示例，这个例子主要演示使用 Go 的 [net](https://golang.org/pkg/net/) 库来创建一个 TCP Server， 并通过 Client 进行连接，最后再通过该连接来收发消息。

- server/main.go

```golang
package main

import (
	"io"
	"log"
	"net"
)

func main() {
    // 创建 TCP 服务，指定监听地址为 127.0.0.1:3000
	l, err := net.Listen("tcp", "127.0.0.1:3000")
	if err != nil {
		log.Fatalf("tcp listen with error: %s \n", err)
	}

	log.Printf("tcp server listen via %s", l.Addr().String())

	for {
        // 等待客户端连接
		c, err := l.Accept()
		if err != nil {
			log.Fatalf("l.Accept(): %v", err)
		}

		log.Printf("tcp connect created \n")

        // 另起 Go 协程处理新建的连接
		go handleConn(c)
	}
}

func handleConn(c net.Conn) {
	defer c.Close()

	// 设置消息体缓冲区大小
	buff := make([]byte, 1024)

	for {
		n, err := c.Read(buff)
		if err != nil && err != io.EOF {
			log.Printf("c.Read(): %v", err)
		}
		if err != nil {
			return
		}

		log.Printf("got request: %s\n", string(buff[:n]))

		// 返回消息
		_, _ = c.Write([]byte("pong"))
	}
}
```

- client/main.go

```go
package main

import (
	"log"
	"net"
	"time"
)

var (
	defaultTimeout = time.Second * 30
)

func main() {
	// 尝试与 baidu.com:80 建立 tcp 连接
	c, err := net.Dial("tcp", "127.0.0.1:3000")

	if err != nil {
		log.Fatal(err)
	}

	defer c.Close() // 退出关闭连接，释放资源

	// 设置写超时时间
	_ = c.SetWriteDeadline(time.Now().Add(defaultTimeout))
	// 发送数据
	_, _ = c.Write([]byte("ping"))

	// 设置读超时时间
	_ = c.SetReadDeadline(time.Now().Add(defaultTimeout))
	//读取数据

	buff := make([]byte, 1024)
	n, _ := c.Read(buff)
	log.Printf("got response: %s\n", buff[:n])
}
```

运行程序，结果如下：

```go
# terminal 1
go run server/main.go

2020/06/02 20:41:45 tcp server listen via 127.0.0.1:3000
2020/06/02 20:41:48 tcp connect created 
2020/06/02 20:41:48 got request: ping
```

```go
# terminal 2
go run client/main.go

2020/06/02 20:41:48 got response: pong
```

参考资料：

- [维基百科](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)
- [Go Net 库](https://golang.org/pkg/net)
- [示例代码](https://github.com/binatify/importgo/tree/master/src/ch10/tcp)