# xml 序列化和反序列化

这一节课将介绍 xml 的序列化和反序列化。

首先看下 xml 标签常见用法：
- `xml:"xxx,omitempty"` 代表如果这个字段值为空，则序列化时忽略该字段
- `xml:"xxx,attr"`      代表该字段为 xml 标签的属性说明 
- `xml:"-"`             代表序列化时忽略该字段
- `xml:"a>b>c"`         代表 xml 标签嵌套模式

以下例子，演示了 
- xml 序列化，包含 xml 标签的不同用法
- 写 xml 文件
- 读 xml 文件
- xml 反序列化

``` go
package main

import (
	"encoding/xml"
	"fmt"
	"io/ioutil"
)

type Student struct {
	Name    string `xml:"name"`              // xml 标签
	Address string `xml:"address,omitempty"` // 如果该字段为空就过滤掉
	Hobby   string `xml:"-"`                 // 进行 xml 序列化的时候忽略该字段
	Father  string `xml:"parent>father"`     // xml 标签嵌套模式
	Mother  string `xml:"parent>mother"`     // xml 标签嵌套模式
	Note    string `xml:"note,attr"`         // xml 标签属性
}

func checkErr(err error) {
	if err != nil {
		panic(err)
	}
}

func main() {
	stu1 := Student{
		Name:  "haha",
		Hobby: "basketball",
	}

	// data, _ := xml.Marshal(s)
	// fmt.Println(string(data))

	// xml 序列化
	newData, err := xml.MarshalIndent(stu1, " ", "    ")
	checkErr(err)
	fmt.Println(string(newData))

	// 写 xml 文件
	err = ioutil.WriteFile("stu.xml", newData, 0644)
	checkErr(err)

	// 读 xml 文件
	content, err := ioutil.ReadFile("stu.xml")
	stu2 := &Student{} 

	// xml 反序列化
	err = xml.Unmarshal(content, stu2) // 注意：第二个参数必须是指针
	checkErr(err)

	fmt.Printf("stu2: %#v\n", stu2)
}
```