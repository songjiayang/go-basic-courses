### 结构体

数组、切片和 Map 可以用来表示同一种数据类型的集合，但是当我们要表示不同数据类型的集合时就需要用到结构体。
> 结构体是由零个或多个任意类型的值聚合成的实体

关键字 `type` 和 `struct` 用来定义结构体：
``` go
type StructName struct{
    FieldName type
}
```

#### 简单示例：定义一个学生结构体
``` go
package main

import "fmt"

type Student struct {
	Age     int
	Name    string
	Address string
}

func main() {
	stu := Student{
		Age:     18,
		Name:    "name",
		Address: "addr",
	}
	fmt.Println(stu)

	// 在赋值的时候，字段名可以忽略
	fmt.Println(Student{20, "new name", "new addr"})

	return
}
```

> 通常结构体中一个字段占一行，但是类型相同的字段，也可以放在同一行，例如：

``` go
type Student struct{
    Age           int
    Name, Address string
}
```

> 一个结构体中的字段名是唯一的，例如一下代码，出现了两个 `Name` 字段，是错误的：

``` go
type Student struct{
    Name string
    Name string
}
``` 

> 结构体中的字段如果是小写字母开头，那么其他 package 就无法直接使用该字段，例如：

``` go
// 在包 pk1 中定义 Student 结构体
package pk1
type Student struct{
    Age  int
    name string
}
```

``` go 
// 在另外一个包 pk2 中调用 Student 结构体
package pk2

func main(){
    stu := Student{}
    stu.Age = 18        //正确
    stu.name = "name"  // 错误，因为`name` 字段为小写字母开头，不对外暴露
}
```


#### 结构体中可以内嵌结构体
> 但是需要注意的是：如果嵌入的结构体是本身，那么只能用指针。请看以下例子。

``` go
package main

import "fmt"

type Tree struct {
	value       int
	left, right *Tree
}

func main() {
	tree := Tree{
		value: 1,
		left: &Tree{
			value: 1,
			left:  nil,
			right: nil,
		},
		right: &Tree{
			value: 2,
			left:  nil,
			right: nil,
		},
	}

	fmt.Printf(">>> %#v\n", tree)
}
```

#### 结构体是可以比较的
> 前提是结构体中的字段类型是可以比较的

``` go
package main

import "fmt"

type Tree struct {
	value       int
	left, right *Tree
}

func main() {
	tree1 := Tree{
		value: 2,
	}

	tree2 := Tree{
		value: 1,
	}

	fmt.Printf(">>> %#v\n", tree1 == tree2)
}
```

#### 结构体内嵌匿名成员
> 声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员

``` go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (p Person) sayHello() {
	fmt.Println("hello")
}

type Student struct {
	Person
}

func main() {
	per := Person{
		Name: "name",
		Age:  18,
	}

	stu := Student{Person: per}

	fmt.Println("stu.Name: ", stu.Name)
	fmt.Println("stu.Age: ", stu.Age)
	stu.sayHello() // 可以直接调用成员的方法
}
```

