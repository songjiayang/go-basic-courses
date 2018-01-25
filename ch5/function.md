### 函数

* 单返回值函数

```go
func plus(a int, b int) (res int){
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

* 递归函数

```go
func recursion(num int) int{
    if num == 0{
        return 1
    }
    return num * recursion(num - 1)
}

func main(){
    // 3 * 2 * 1
    fmt.Println(recursion(3))
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
func addInt() func() int {
     i := 0
     return func() int {
       i += 1
       return i
     }
}

func main(){
     num := addInt()
     fmt.Println(num())
     fmt.Println(num())
     fmt.Println(num())
}
```
