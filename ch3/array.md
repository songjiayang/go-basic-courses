### 数组

* 定义：由若干相同类型的元素组成的序列
* 数组的长度是固定的，声明后无法改变
* 数组的长度是数组类型的一部分，eg：元素类型相同但是长度不同的两个数组是不同类型的
* 需要严格控制程序所使用内存时，数组十分有用，因为其长度固定，避免了内存二次分配操作

#### eg1:显示定义数组长度

```go
package main

import "fmt"

func main() {
	// 定义长度为 5 的数组
	var arr1 [5]int
	for i := 0; i < 5; i++ {
		arr1[i] = i
	}
	printHelper("arr1", arr1)

	// 以下赋值会类型不匹配错误，因为数组长度是数组类型的一部分
	// arr1 = [3]int{1, 2, 3}
	arr1 = [5]int{2, 3, 4, 5, 6} // 长度和元素类型都相同，可以正确赋值

	// 简写模式，在定义的同时给出了赋值
	arr2 := [5]int{0, 1, 2, 3, 4}
	printHelper("arr2", arr2)

	// 数组元素类型相同并且数组长度相等的情况下，数组可以进行比较
	fmt.Println(arr1 == arr2)

	// 也可以不显式定义数组长度，由编译器完成长度计算
	var arr3 = [...]int{0, 1, 2, 3, 4}
	printHelper("arr3", arr3)

	// 定义前四个元素为默认值 0，最后一个元素为 -1
	var arr4 = [...]int{4: -1}
	printHelper("arr4", arr4)

	// 多维数组
	var twoD [2][3]int
	for i := 0; i < 2; i++ {
		for j := 0; j < 3; j++ {
			twoD[i][j] = i + j
		}
	}
	fmt.Println("twoD: ", twoD)
}

func printHelper(name string, arr [5]int) {
	for i := 0; i < 5; i++ {
		fmt.Printf("%v[%v]: %v\n", name, i, arr[i])
	}

	// len 获取长度
	fmt.Printf("len of %v: %v\n", name, len(arr))

	// cap 也可以用来获取数组长度
	fmt.Printf("cap of %v: %v\n", name, cap(arr))

	fmt.Println()
}
```
