# 方法

方法主要源于 OOP 语言，在传统面向对象语言中 (例如 C++), 我们会用一个“类”来封装属于自己的数据和函数，这些类的函数就叫做方法。

虽然 Go 不是经典意义上的面向对象语言，但是我们可以在一些接收者（自定义类型，结构体）上定义函数，同理这些接收者的函数在 Go 里面也叫做方法。


### 声明

方法（method）的声明和函数很相似, 只不过它必须指定接收者。

```
func (t T) F() {}
func (t *T) N() {}
```

注意：

 - 接收者指凡是用关键字 `type` 定义的类型，例如自定义类型，结构体。
 - 同一个接收者的方法名不能重复 (没有重载)。
 - 结构体中方法名不能和字段重复。

### 自定义类型添加方法

```
package main

import "fmt"

type Age int

func (a Age) PrintAge() {
	fmt.Println("Your age is", a)
}

func main() {
	a := Age(18)
	a.PrintAge()
}

```

### 结构体添加方法

```
package main

import "fmt"

type Student struct {
	Name string

	Age int
}

func (s *Student) AddAge(year int) {
	s.Age += year
}

func (s *Student) PrintAge() {
	fmt.Printf("%s's age is %d\n", s.Name, s.Age)
}

func main() {
	s := Student{"Tony", 8}
	s.PrintAge()

	s.AddAge(6)
	s.PrintAge()
}
```

### 值和指针作为接收者的区别

```
package main

import "fmt"

type T struct {
	a int
}

func (m T) StayTheSame() {
	m.a = 3
}

func (m *T) Update() {
	m.a = 3
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

结论： 使用值作为接收者（`T`） 不会修改结构体值，而指针 `*T` 可以修改。
