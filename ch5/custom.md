# 自定义类型

前面我们已经学习了不少基础和高级数据类型，在 Go 语言里面，我们还可以通过自定义类型来表示一些特殊的数据结构和业务逻辑。

使用关键字 `type` 来声明：

```
type NAME TYPE
```

### 声明语法

- 单次声明

```
type City string
```

- 批量声明

```
type (
    B0 = int8
    B1 = int16
    B2 = int32
    B3 = int64
)

type (
    A0 int8
    A1 int16
    A2 int32
    A3 int64
)
```

### 简单示例

```
package main

import "fmt"

type City string

func main() {
    city := City("上海")
    fmt.Println(city)
}

```

### 基本操作

```
package main

import "fmt"

type City string
type Age int

func main() {
    city := City("北京")
    fmt.Println("I live in", city + " 上海")  //  字符串拼接
    fmt.Println(len(city))  // len 方法

    middle := Age(12)

    if middle >= 12 {
        fmt.Println("Middle is bigger than 12")
    }
}
```

总结： 自定义类型的原始类型的所有操作同样适用。

### 函数参数

```
package main

import "fmt"

type Age int

func main() {
    middle := Age(12)
    printAge(middle)
}

func printAge(age int) {
    fmt.Println("Age is", age)
}
```

当我们运行代码的时候会出现  `./main.go:11:10: cannot use middle (type Age) as type int in argument to printAge` 的错误。

因为 `printAge` 方法期望的是 int 类型，但是我们传入的参数是 `Age`，他们虽然具有相同的值，但为不同的类型。

我们可以采用显式的类型转换（ `printAge(int(primary))`）来修复。

### 不同自定义类型间的操作

```
package main

import "fmt"

type Age int
type Height int

func main() {
    age := Age(12)
    height := Height(175)

    fmt.Println(height / age)
}
```

当我们运行代码会出现 `./main.go:12:21: invalid operation: height / age (mismatched types Height and Age)` 错误，修复方法使用显式转换:

```
fmt.Println(int(height) / int(age))
```
