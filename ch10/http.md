# HTTP

上一章介绍了 Socket 的使用，主要提到了 TCP 以及 UDP，它们是属于传输层的协议，今天我们将介绍一个非常流出的应用层协议--HTTP。

### 为什么是 HTTP

[HTTP](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE) 是建立在 TCP 之上的
超文本传输协议，是万维网的数据通信基础。

传统的 HTTP 协议是基于 Request/Response 模式的，一般一次请求会新建一个连接，响应结束会关闭连接。这对于早起服务端贫瘠的资源非常有意义，
使其能够尽快释放连接资源，服务更多的用户。

HTTP 能够流行起来，最主要一点还是因为它对于用户而言，使用起来简单方便。

Go 语言的标准库提供了 HTTP 完整的支持，这点比很多语言好太多，你可以不使用任何第三方库或者框架就能轻松写出一个的 HTTP Server 或者 HTTP Client。

下面我们就来看一个最简单的例子：

#### HTTP Server

```go
# go-example/server/
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

使用命令 `go run main`

### HTTP Server over TLS

### HTTP Client
