# channel

### channel 简介
goroutine 是 Go 中实现并发的重要机制，channel 是 goroutine 之间进行通信的重要桥梁。

使用内建函数 make 可以创建 channel，举例如下：
``` go
ch := make(chan int)  // 注意： channel 必须定义其传递的数据类型
```

也可以用 var 声明 channel, 如下： 
``` go
var ch chan int
``` 

以上声明的 channel 都是双向的，意味着可以该 channel 可以发送数据，也可以接收数据。

“发送”和“接收”是 channel 的两个基本操作。
```go 
ch <- x // channel 接收数据 x

x <- ch // channel 发送数据并赋值给 x

<- ch // channel 发送数据，忽略接受者
```

### channel buffer

上文提到，可以通过  `make(chan int)` 创建channel，此类 channel 称之为非缓冲通道。事实上 channel 可以定义缓冲大小，如下：
 
``` go
chInt := make(chan int)       // unbuffered channel  非缓冲通道
chBool := make(chan bool, 0)  // unbuffered channel  非缓冲通道
chStr := make(chan string, 2) // bufferd channel     缓冲通道
```

需要注意的是，程序中必须同时有不同的 goroutine 对非缓冲通道进行发送和接收操作，否则会造成阻塞。

以下是一个错误的使用示例：
``` go
func main() {
	ch := make(chan string)
	
	ch <- "ping"
	
	fmt.Println(<-ch)
}
```
这一段代码运行后提示错误： `fatal error: all goroutines are asleep - deadlock!`。

因为 main 函数是一个 goroutine, 在这一个 goroutine 中发送了数据给非缓冲通道，但是却没有另外一个 goroutine 从非缓冲通道中里读取数据，
所以造成了阻塞或者称为死锁。

在以上代码中添加一个 goroutine 从非缓冲通道中读取数据，程序就可以正常工作。如下所示： 
``` go
func main() {
	ch := make(chan string)

	go func() {
		ch <- "ping"
	}()

	fmt.Println(<-ch)
}
```

与非缓冲通道不同，缓冲通道可以在同一个 goroutine 内接收容量范围内的数据，即便没有另外的 goroutine 进行读取操作，如下代码可以正常执行：
``` go
func main() {
	ch := make(chan int, 2)
	
	ch <- 1
	
	ch <- 2
}
```

### 单向 channel

单向通道即限定了该 channel 只能接收或者发送数据，单向通道通常作为函数的参数，如下例所示： 
``` go
func receive(receiver chan<- string, msg string) {
	receiver <- msg
}

func send(sender <-chan string, receiver chan<- string) {
	msg := <-sender
	receiver <- msg
}

func main() {
	ch1 := make(chan string, 1)
	ch2 := make(chan string, 1)

	receive(ch1, "pass message")
	send(ch1, ch2)
	
	fmt.Println(<-ch2)
}
```

需要注意的是，在变量声明中是不应该出现单向通道的，因为通道本来就是为了通信而生，只能接收或者只能发送数据的通道是没有意义的。
请看下面这个例子：
``` go
func main() {
	ch := make(chan <- string , 1)
	ch <- "str"
}
```
这个例子中定义了一个只能用来接收数据的通道，从语法上来看没有错误，但这是一种糟糕的实践。


### select 语句

select 专门用于通道发送和接收操作，看起来和 switch 很相似，但是进行选择和判断的方法完全不同。

在下述例子中，通过 select 的使用，保证了 worker 中的事务可以执行完毕后才退出 main 函数
``` go
func worker(ch chan string) {
	fmt.Println("do something...")
	ch <- "str"
}

func main() {
	ch := make(chan string)

	go worker(ch)

	select {
	case <-ch:
		fmt.Println("get value from chan")
	}
}
```
> 思考： 如果上述例子中，没有这个 select ，那么 worker 函数是否有机会执行？

### 通过 channel 实现同步机制
一个经典的例子如下，main 函数中起了一个 goroutine，通过非缓冲队列的使用，能够保证在 goroutine 执行结束之前 main 函数不会提前退出。

``` go
func worker(done chan bool){
	fmt.Println("start working...")
	done <- true
	fmt.Println("end working...")
}

func main() {
    done := make(chan bool, 1)

    go worker(done)

    <- done
}
```

思考：如果把上述例子中的 `<-done` 注释掉，运行结果会如何？
