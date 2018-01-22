### 切片

#### 切片组成要素：

* 指针：指向底层数组
* 长度：切片中元素的长度，不能大于容量
* 容量：指针所指向的底层数组的总容量

#### 常见初始化方式

* 使用 `make` 初始化

```go
slice := make([]int, 5)     // 初始化长度和容量都为 5 的切片
slice := make([]int, 5, 10) // 初始化长度为 5, 容量为 10 的切片
```

* 使用简短定义

```go
slice := []int{1, 2, 3, 4, 5}
```

* 使用数组来初始化切片

```go
arr := [5]int{1, 2, 3, 4, 5}
slice := arr[0:3] // 左闭右开区间，最终切片为 [1,2,3]
```

* 使用切片来初始化切片

```go
sliceA := []int{1, 2, 3, 4, 5}
sliceB := sliceA[0:3] // 左闭右开区间，sliceB 最终为 [1,2,3]
```

#### 长度和容量

```go
package main

import (
	"fmt"
)

func main() {
	slice := []int{1, 2, 3, 4, 5}
	fmt.Println("len: ", len(slice))
	fmt.Println("cap: ", cap(slice))

	//改变切片长度
	slice = append(slice, 6)
	fmt.Println("after append operation: ")
	fmt.Println("len: ", len(slice))
	fmt.Println("cap: ", cap(slice)) //注意，底层数组容量不够时，会重新分配数组空间，通常为两倍
}
```

以上代码，预期输出如下：

```
len:  5
cap:  5
after append operation:
len:  6
cap:  12
```

#### 注意点

* 多个切片共享一个底层数组的情况

> 对底层数组的修改，将影响上层多个切片的值

```go
package main

import (
	"fmt"
)

func main() {
	slice := []int{1, 2, 3, 4, 5}
	newSlice := slice[0:3]
	fmt.Println("before modifying underlying array:")
	fmt.Println("slice: ", slice)
	fmt.Println("newSlice: ", newSlice)
	fmt.Println()

	newSlice[0] = 6
	fmt.Println("after modifying underlying array:")
	fmt.Println("slice: ", slice)
	fmt.Println("newSlice: ", newSlice)
}
```

以上代码预期输出如下：

```
before modify underlying array:
slice:  [1 2 3 4 5]
newSlice:  [1 2 3]

after modify underlying array:
slice:  [6 2 3 4 5]
newSlice:  [6 2 3]
```

* 使用 `copy` 方法可以避免共享同一个底层数组

> 示例代码如下：

```go
package main

import (
	"fmt"
)

func main() {
	slice := []int{1, 2, 3, 4, 5}
	newSlice := make([]int, len(slice))
	copy(newSlice, slice)
	fmt.Println("before modifying underlying array:")
	fmt.Println("slice: ", slice)
	fmt.Println("newSlice: ", newSlice)
	fmt.Println()

	newSlice[0] = 6
	fmt.Println("after modifying underlying array:")
	fmt.Println("slice: ", slice)
	fmt.Println("newSlice: ", newSlice)
}
```

以上代码预期输出如下：

```
before modifying underlying array:
slice:  [1 2 3 4 5]
newSlice:  [1 2 3 4 5]

after modifying underlying array:
slice:  [1 2 3 4 5]
newSlice:  [6 2 3 4 5]
```

### 小练习

如何使用 `copy` 函数进行切片部分拷贝？

```go
// 假设切片 slice 如下:
slice := []int{1, 2, 3, 4, 5}

// 如何使用 copy 创建切片 newSlice, 该切片值为 [2, 3, 4]
newSlice = copy(?,?)
```
