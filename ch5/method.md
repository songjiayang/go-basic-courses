# 方法

方法主要源于 OOP 语言，在传统面向对象语言中 (例如 C++), 我们会用一个“类”来封装属于自己的数据和函数，这些类的函数就叫做方法。

虽然 Go 不是经典意义上的面向对象语言，但是我们可以在一些接受者（自定义类型，结构体）上定义函数，同理这些接受者的函数在 Go 里面也叫做方法。


### 和函数对比

前面我们已经讲过函数(function), 方法 (method) 和函数的主要区别为：

- 函数属于整个包，没有接收者。
- 方法属于对应的接受者。

更多资料和讨论请参考[链接](https://stackoverflow.com/questions/8263546/whats-the-difference-of-functions-and-methods-in-go)。


### 自定义类型添加方法

```
package main

import "fmt"

type Collection []int

func (c Collection) Sum() int {
	sum := 0
	for _, item := range c {
		sum += item
	}

	return sum
}

func main() {
	c := Collection{
		1, 2, 3, 4, 5, 6, 7, 8, 9,
	}

	fmt.Println("Sum is", c.Sum())
}
```

### 结构体添加方法

```
package main

import "fmt"

type Student struct {
	Age  int
	Name string
}

func (s *Student) Print() {
	fmt.Println(s.Age)
	if s.Age <= 12 {
		fmt.Println(s.Name, "是一名小学生")
	} else {
		fmt.Println(s.Name, "不是一名小学生")
	}
}

func (s *Student) GrowUp(year int) {
	s.Age += year
}

func main() {
	s := &Student{8, "小明"}
	s.Print()

	s.GrowUp(6)
	s.Print()
}
```
