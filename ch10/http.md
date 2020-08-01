# HTTP

上一章介绍了 Socket 的使用，主要提到了 TCP 以及 UDP，它们是属于传输层的协议，今天我们将介绍一个非常流出的应用层协议--HTTP。

## 为什么是 HTTP

[HTTP](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE) 是建立在 TCP 之上的
超文本传输协议，是万维网的数据通信基础。

传统的 HTTP 协议是基于 Request/Response 模式的，一般一次请求会新建一个连接，响应结束会关闭连接。这对于早起服务端贫瘠的资源非常有意义，使其能够尽快释放连接资源，服务更多的用户。

HTTP 能够流行起来，最主要一点还是因为它对于用户而言，使用起来简单方便。

Go 语言的标准库提供了 HTTP 完整的支持，这点比很多语言好太多，你可以不使用任何第三方库或者框架就能轻松写出一个的 HTTP Server 或者 HTTP Client。

## HTTP Server

下面我们就来看一个最简单的例子。

```go
// http/server/example01.go

package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w,
			"Hello %s, current time is %s \n",
			r.URL.Query().Get("name"),
			time.Now().Format(time.RFC3339))
	})

	log.Fatal(http.ListenAndServe(":3000", nil))
}

```

本示例中我主要使用 Go 的 `http` 包新建并在 3000 端口启动一个 HTTP Server，然手我向这个 Server 注册一个 path 为 `/` 的处理函数。

处理函数逻辑为读取 HTTP Request 中的 `name` Query 参数，然后向 HTTP Response 的 Body 打印输出获取的 `name` 和当前时间。运行效果如下：

```bash
$ curl http://localhost:3000?name=songjiayang
Hello songjiayang, current time is 2020-08-01T21:57:39+08:00 
```

代码说明：

- `ListenAndServe(addr string, handler Handler) error` 该方法可以新建和启动一个 HTTP Server，它主要包括两个参数，第一个是监听地址，第二个是 `http.Handler`。 如果 handler 为 nil， 那么会使用 http.DefaultServeMux 这个 默认 Handler。
- `HandleFunc(pattern string, handler func(ResponseWriter, *Request))` 该方法就是向 http.DefaultServeMux 这个 默认 Handler 注册对应的处理逻辑，而默认的 DefaultServeMux 内部其实是使用一个 map 来存储这些 path 的注册的，不支持正则。

##### 配置 Http Server

上面的例子中我们使用 `http.ListenAndServe` 直接完成了 Server 的创建和监听，但是如果我们想做一些配置，例如 读写超时时间、TLS ，那么就需要将 Server 的创建和 Listen 分开，代码如下：

```go
// http/server/example02.go

package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w,
			"Hello %s, current time is %s \n",
			r.URL.Query().Get("name"),
			time.Now().Format(time.RFC3339))
	})

	server := &http.Server{
		Addr: ":3000",
		Handler: nil, 
		TLSConfig: nil, 
		ReadTimeout: 0, 
		ReadHeaderTimeout: 0, 
		WriteTimeout: 0,
		IdleTimeout: 0,
		// 其它字段 ..
	}

	log.Fatal(server.ListenAndServe())
}

```

字段解读：

- Handler:  具体的 handler，如果为空则使用默认的 http.DefaultServeMux
- TLSConfig: TLS 相关的配置，如果不为空，那么使用 TLS 监听
- ReadTimeout: 整个 http request 的读超时时间，包括 header 和 body，如果为 0 表示永远不超时
- ReadHeaderTimeout: http request header 读取超时时间设置，如果为 0 表示永远不超时
- WriteTimeout: http response 写超时时间，如果为 0 表示永远不超时
- IdleTimeout: keep alive 的 http 连接空闲超时时间，如果为空 0 表示永远不超时


##### 自定义 http.Handler

前面的例子我们使用的都是 `http.DefaultServeMux` 这个 http 包默认的 Handler，那么我们如何实现一个属于自己的 Handler 呢？

http 包中的 Handler 是一个接口，定义如下：

```
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

所以只要实现该接口即可，下面我们就编写一个自己的 Handler 来重写上面的代码逻辑：

```golang
// http/server/example03.go

package main

import (
	"fmt"
	"log"
	"net/http"
	"sync"
	"time"
)

func main() {
	handler := NewMyHandler()
	handler.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w,
			"Hello %s, current time is %s \n",
			r.URL.Query().Get("name"),
			time.Now().Format(time.RFC3339))
	})

	handler.HandleFunc("/ping", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprint(w, "Pong\n")
	})

	log.Fatal(http.ListenAndServe(":3000", handler))
}

type MyHandler struct {
	mux    sync.RWMutex
	router map[string]HandFunc
}

type HandFunc func(w http.ResponseWriter, r *http.Request)

func NewMyHandler() *MyHandler {
	return &MyHandler{
		router: make(map[string]HandFunc, 8),
	}
}

func (h *MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	h.mux.RLock()
	hf, ok := h.router[r.URL.Path]
	h.mux.RUnlock()

	// if not found
	if !ok {
		http.NotFound(w, r)
		return
	}

	hf(w, r)
}

func (h *MyHandler) HandleFunc(pattern string, hf HandFunc) {
	h.mux.Lock()
	h.router[pattern] = hf
	h.mux.Unlock()
}
```

代码运行结果如下：

```
$ curl http://localhost:3000?name=songjiayang
Hello songjiayang, current time is 2020-08-01T22:59:28+08:00 

curl http://localhost:3000/ping
Pong
```

### HTTP Client
