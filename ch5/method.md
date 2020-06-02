# 方法

方法主要源于 OOP 语言，在传统面向对象语言中 (例如 C++), 我们会用一个“类”来封装属于自己的数据和函数，这些类的函数就叫做方法。

虽然 Go 不是经典意义上的面向对象语言，但是我们可以在一些接收者（自定义类型，结构体）上定义函数，同理这些接收者的函数在 Go 里面也叫做方法。


### 声明

方法（method）的声明和函数很相似, 只不过它必须指定接收者：

```golang
func (t T) F() {}
```

注意：

- 接收者的类型只能为用关键字 `type` 定义的类型，例如自定义类型，结构体。
- 同一个接收者的方法名不能重复 (没有重载)，如果是结构体，方法名还不能和字段名重复。
- 值作为接收者无法修改其值，如果有更改需求，需要使用指针类型。

### 简单例子

```golang
package main

type T struct{}

func (t T) F()  {}

func main() {
	t := T{}
	t.F()
}
```

### 接收者类型不是任意类型

例如：

```golang
package main

func (t int64) F()  {}

func main() {
	t := int64(10)
	t.F()
}
```

当运行以下代码会得到 `cannot define new methods on non-local type int64` 类似错误信息，我们可以使用自定义类型来解决：

```golang
package main

type T int64
func (t T) F()  {}

func main() {
	t := T(10)
	t.F()
}
```

> 小结：接收者不是任意类型，它只能为用关键字 `type` 定义的类型（例如自定义类型，结构体）。

### 命名冲突

a. 接收者定义的方法名不能重复, 例如：

```golang
package main

type T struct{}

func (T) F()         {}
func (T) F(a string) {}

func main() {
	t := T{}
	t.F()
}
```

运行代码我们会得到 `method redeclared: T.F` 类似错误。

b. 结构体方法名不能和字段重复，例如：

```golang
package main

type T struct{
  F string
}

func (T) F(){}

func main() {
	t := T{}
	t.F()
}
```

运行代码我们会得到 `: type T has both field and method named F` 类似错误。

> 小结： 同一个接收者的方法名不能重复 (没有重载)；如果是结构体，方法名不能和字段重复。

### 接收者可以同时为值和指针

在 Go 语言中，方法的接收者可以同时为值或者指针，例如：

```golang
package main

type T struct{}

func (T) F()  {}
func (*T) N() {}

func main() {
	t := T{}
	t.F()
	t.N()

	t1 := &T{} // 指针类型
	t1.F()
	t1.N()
}
```

可以看到无论值类型 `T` 还是指针类型 `&T` 都可以同时访问 `F` 和  `N` 方法。

### 值和指针作为接收者的区别

同样我们先看一段代码：


```golang
package main

import "fmt"

type T struct {
	value int
}

func (m T) StayTheSame() {
	m.value = 3
}

func (m *T) Update() {
	m.value = 3
}

func main() {
	m := T{0}
	fmt.Println(m) // {0}

	m.StayTheSame()
	fmt.Println(m) // {0}

	m.Update()
	fmt.Println(m) // {3}
}
```

运行代码输出结果为：

```golang
{0}
{0}
{3}
```

> 小结：值作为接收者（`T`） 不会修改结构体值，而指针 `*T` 可以修改。
