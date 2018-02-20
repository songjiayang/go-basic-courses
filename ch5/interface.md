### 接口

接口类型是一种抽象类型，是方法的集合，其他类型实现了这些方法就是实现了这个接口。

``` go
/* 定义接口 */
type interface_name interface {
   method_name1 [return_type]
   method_name2 [return_type]
   method_name3 [return_type]
   ...
   method_namen [return_type]
}
```

#### 简单示例：打印不同几何图形的面积和周长
``` go
package main

import (
	"fmt"
	"math"
)

type geometry interface {
	area() float32
	perim() float32
}

type rect struct {
	len, wid float32
}

func (r rect) area() float32 {
	return r.len * r.wid
}

func (r rect) perim() float32 {
	return 2 * (r.len + r.wid)
}

type circle struct {
	radius float32
}

func (c circle) area() float32 {
	return math.Pi * c.radius * c.radius
}

func (c circle) perim() float32 {
	return 2 * math.Pi * c.radius
}

func show(name string, g geometry) {
	fmt.Printf("area of %v is %v \n", name, g.area())
	fmt.Printf("perim of %v is %v \n", name, g.perim())
}

func main() {
	rec := rect{
		len: 1,
		wid: 2,
	}
	show("rect", rec)

	cir := circle{
		radius: 1,
	}
	show("circle", cir)

	return
}
```