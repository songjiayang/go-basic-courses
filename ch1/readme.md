# Go 安装和配置

### 写在前面

- 教学 Go 版本 1.9.x
- 教学使用 `GOPATH` 为 `~/importgo`

### 下载并安装

- 下载安装对应版本 https://golang.org/dl/
- 查看 go 安装目录 `/usr/local/go` (Windows 下默认安装到 `c:\Go`)
- 运行 `go version` 命令检查是否安装正确

>> 推荐大家使用二进制默认安装方式

### 项目环境变量

本课程命名为 `importgo`, 故添加一个 `IMPORTGOROOT` 的环境变量进行所有的代码开发和演示，具体配置如下：


```
$ vi ~/.profile

export IMPORTGOROOT=$HOME/importgo
export GOPATH=$IMPORTGOROOT # 覆盖 GOPATH 环境变为 importgo
export PATH=$IMPORTGOROOT/bin:$PATH #
```

当我们配置完毕后，可以执行 `source ~/.profile` 更新系统环境变量。

### 编写我的第一个 Go 程序

首先需要在 GOPATH 文件夹下创建一个 `src` 目录用于存放我们的源代码。

```
$ mkdir -p $GOPATH/src
```

然后我们在 src 目录下面新建 `hello/hello.go` 的文件，内容如下：

```golang
package main

import "fmt"

func main() {
    fmt.Println("hello, world")
}
```

我们使用 `go run hello.go` 来运行程序，输出为：

```
hello, world
```

### Go 命令详解

- go run: 运行 Go 源码程序
- go build: 编译 Go 源码
- go install: Go 源码编译并打包到 $GOPATH/bin 目录