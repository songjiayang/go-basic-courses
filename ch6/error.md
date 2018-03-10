# 异常处理

### defer
上一节课介绍了常见的流程控制，这一节课将介绍一个 Go 特有的流程控制语句： defer


defer 通常用于延迟调用指定的函数。例如：

``` go
func outerFunc() {
    defer fmt.Println(" World!")

    fmt.Println("Hello")
}
```

上例最终输出的结果是： "Hello World!". 

这是因为：`defer` 会在 `outerFunc` 退出之前执行打印操作，因此被 `defer` 调用的函数也称为“延迟函数”。


##### defer 常用场景

defer语句经常被用于处理成对的操作，如打开和关闭，连接和断开连接，加锁和释放锁。恰当使用 defer 能够保证资源正确释放。
以下是几个例子：
``` go
// 使用 defer 关闭 http 请求响应体的 Body
func closeBody(url string) error {
    resp, err := http.Get(url)
    defer resp.Body.Close()
    // ... do more stuff ...

    return err
}
```

``` go
// 使用 defer 关闭文件句柄
func closeFile(filename string) error{
    f, err := os.Open(filename)
    defer f.Close()
    // ... do more stuff ...

    return err
}
```

``` go
// 使用 defer 解锁
func BillCustomer(c *Customer)  {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    // ... do more stuff ...

    return 
}
```

##### defer 使用中一些注意点
请看以下例子，猜下输出结果是？

``` go
func printNumber() {
    for i := 0; i<5; i++{
        defer func(){
            fmt.Println(i)
        }()
    }
}
```

最终输出的结果是 `5 5 5 5 5`。 这是因为 defer 所调用的函数是延迟执行的。等到执行 defer 所调用的函数时，i 已经是 5 了。
接着看下面这个例子:
``` go
func printNum() {
	for i := 0; i < 5; i++ {
		defer func(v int) {
			fmt.Println(v)
		}(i)
	}
}
```
这个例子最终输出的是： `4 3 2 1 0`。具体是什么原因留作大家思考。

### panic
当程序遇到致命错误导致无法继续运行时就会出发 `panic` , 例如：数组越界，空指针等。

以下代码将会出发数组越界异常。
``` go
s := []int{1, 2, 3}
for i := 0; i <= 4; i++ {
	fmt.Println(s[i])
}
```
上述例子是被动触发了 `panic` 事件。在实际开发中，也可以主动调用 `panic`,调用后会停止当前控制流程并引发一个运行时恐慌。
``` go
func panicFunc() {
	panic(errors.New("this is test for panic"))
}
```
### recover
顾名思义，recover 函数能使当前程序从 panic 中恢复。recover 能够拦截 panic 事件，使得程序不会因为意外而触发 panic 事件而完全退出。

recover 函数返回的是一个 interfac{} 类型的结果，如果捕获到了 panic 事件，该结果就为非 nil。见下例：

``` go
func panicFunc() {
	defer func() {
		if p := recover(); p != nil {
			fmt.Println("recover panic")
		}
	}()

	panic(errors.New("this is test for panic"))
}
func main() {
	fmt.Println("before panic")

	panicFunc()

	fmt.Println("after panic")
}

```


### 课后练习
- 思考以下代码的输出，并解释为什么

``` go
fund printNumbers(){
    for i:=0; i<5; i++{
        defer func(n int){
            fmt.Printf(n)
        }(i * 2)
    }
}
```
