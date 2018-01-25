### 函数

> 函数是语句序列的集合，能够将一个大的工作分解为小的任务，对外隐藏了实现细节

* 函数组成
  * 函数名
  * 参数列表(parameter-list)
  * 返回值(result-list)
  * 函数体(body)

```go
func name(parameter-list) (result-list){
    body
}
```

* 单返回值函数

```go
func plus(a, b int) (res int){
	return a + b
}
```

* 多返回值函数

```go
func multi()(string, int){
    return "name", 18
}
```

* 命名返回值

```go
// 被命名的返回参数的值为该类型的默认零值
// 该例子中 name 默认初始化为空字符串，height 默认初始化为 0
func namedReturnValue()(name string, height int){
    name = "xiaoming"
    height = 180
    return
}
```

* 参数可变函数

```go
func sum(nums ...int)int{
    fmt.Println("len of nums is : ", len(nums))
    res := 0
    for _, v := range nums{
        res += v
    }
    return res
}

func main(){
    fmt.Println(sum(1))
    fmt.Println(sum(1,2))
    fmt.Println(sum(1,2,3))
}
```

* 匿名函数

```go
func main(){
    func(name string){
       fmt.Println(name)
    }("禾木课堂")
}
```

* 闭包

```go
func addOne() func() int {
     i := 0
     return func() int {
       i += 1
       return i
     }
}

func main(){
     num := addOne()
     fmt.Println(num())
     fmt.Println(num())
     fmt.Println(num())
}
```

* 函数作为参数

```go
func sayHello(name string) {
	fmt.Println("Hello ", name)
}

func logger(f func(string), name string) {
	fmt.Println("start calling method sayHello")
	f(name)
	fmt.Println("end calling method sayHellog")
}

func main() {
	logger(sayHello, "禾木课堂")
}
```

* 传值和传引用

```go
func sendValue(name string) {
	name = "hemuketang"
}

func sendAddress(name *string) {
	*name = "hemuketang"
}

func main() {
	// 传值和传引用
	str := "禾木课堂"
	fmt.Println("before calling sendValue, str : ", str)
	sendValue(str)
	fmt.Println("after calling sendValue, str : ", str)

	fmt.Println("before calling sendAddress, str : ", str)
	sendAddress(&str)
	fmt.Println("after calling sendAddress, str: ", str)
}
```
