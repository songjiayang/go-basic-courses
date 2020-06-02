# JSON 序列化和反序列化

上一节我们介绍了 `xml` 的使用，其实对于数据的序列化和反序列化还有一种更为常见的方式，那就是 [JSON](https://www.json.org/json-zh.html)，尤其是在 http, rpc 的微服务调用中。

### 基础语法

在 Go 中我们主要使用官方的 [encoding/json](https://golang.org/pkg/encoding/json/) 包对 JSON 数据进行序列化和反序列化，主要使用方法有：

- 序列化:
```golang
func Marshal(v interface{}) ([]byte, error)
```

- 反序列化：
```golang
func Unmarshal(data []byte, v interface{}) error
```

简单例子：

```golang
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	var (
		data  = `1`
		value int
	)

	err1 := json.Unmarshal([]byte(data), &value)

	fmt.Println("Unmarshal error is:", err1)
	fmt.Printf("Unmarshal value is: %T, %d \n", value, value)

	value2, err2 := json.Marshal(value)

	fmt.Println("Marshal error is:", err2)
	fmt.Printf("Marshal value is: %s \n", string(value2))
}
```

当我们运行代码的时候可以得到如下输出结果：

```
Unmarshal error is: <nil>
Unmarshal value is: int, 1 
Marshal error is: <nil>
Marshal value is: 1 
```

在这个例子中，我们使用 `Unmarshal` 和 `Marshal` 将一个整数的 JSON 二进制转化为 go `int` 数据。

>> 注意：在实际应用中，我们在序列化和反序列化的时候，需要检查函数返回的 err, 如果 err 不为空，表示数据转化失败。

例如：我们把上面例子中 `value` 类型由 `int` 修改为 `string` 后再次运行代码，你将得到 `Unmarshal error is: json: cannot unmarshal number into Go value of type string` 的错误提醒。

### 数据对应关系

JSON 和 Go 数据类型对照表：

|类型| JSON        | Go         |
|:-------------:| :-------------: |:-------------:| 
|bool| true, false | true, false|
|string| "a"| string("a")|
|整数|    1 | int(1), int32(1), int64(1) ...|
|浮点数| 3.14 | float32(3.14), float64(3.14) ...| 
|数组| [1,2]| [2]int{1,2}, []int{1, 2}|
|对象 Object|{"a": "b"}| map[string]string, struct|
|未知类型|...| interface{}|

例如：

```golang
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	var (
		d1 = `false`
		v1 bool
	)

	json.Unmarshal([]byte(d1), &v1)
	printHelper("d1", v1)

	var (
		d2 = `2`
		v2 int
	)

	json.Unmarshal([]byte(d2), &v2)
	printHelper("d2", v2)

	var (
		d3 = `3.14`
		v3 float32
	)

	json.Unmarshal([]byte(d3), &v3)
	printHelper("d3", v3)

	var (
		d4 = `[1,2]`
		v4 []int
	)

	json.Unmarshal([]byte(d4), &v4)
	printHelper("d4", v4)

	var (
		d5 = `{"a": "b"}`
		v5 map[string]string
		v6 interface{}
	)

	json.Unmarshal([]byte(d5), &v5)
	printHelper("d5", v5)

	json.Unmarshal([]byte(d5), &v6)
	printHelper("d5(interface{})", v6)
}

func printHelper(name string, value interface{}) {
	fmt.Printf("%s Unmarshal value is: %T, %v \n", name, value, value)
}
```
运行代码我们可以得到如下输出结果：

```bash
d1 Unmarshal value is: bool, false 
d2 Unmarshal value is: int, 2 
d3 Unmarshal value is: float32, 3.14 
d4 Unmarshal value is: []int, [1 2] 
d5 Unmarshal value is: map[string]string, map[a:b] 
d5(interface{}) Unmarshal value is: map[string]interface {}, map[a:b] 
```

### 自定义数据类型

除了使用上面基础数据外，对于那些比较复杂的数据集合（Object），我们还可以使用自定义数据类型 struct 来转化。

- Go 中关于 JSON 转化字段名的对应语法为：

```golang
Field int `json:"myName"`
```

- 如果我们想忽略那些空值的字段，我们可以使用 `omitempty` 选项：

```golang
Field int `json:"myName,omitempty"`
```

- 如果我们想忽略特定字段:

```golang
Field int `json:"-"`
```
组合示例：

```
type A struct {
	A int     `json:"k"`
	B string  `json:"b,omitempty"`
	C float64 `json:"-"`
}
```

### 实际例子练习

假如我们有这样一段 JSON 数据，它表示一个学生的考试成绩，下面我们就来看看在 Go 中如何序列化和反序列化。

- 数据准备:

```
# data.json
{
    "id": 1,
    "name": "小红",
    "results": [
        {
            "name": "语文",
            "score": 90
        },
        {
            "name": "数学",
            "score": 100
        }
    ]
}
```

- 反序列化：

```golang
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
)

type Result struct {
	Name  string  `json:"name"`
	Score float64 `json:"score"`
}

type Student struct {
	Id int `json:"id"`

	Name    string   `json:"name"`
	Results []Result `json:"results"`
}

func main() {
	dat, _ := ioutil.ReadFile("data.json")

	var s Student
	json.Unmarshal(dat, &s)
	fmt.Printf("Student's result is: %v\n", s)
}
```

运行代码输出结果为：

```
Student's result is: {1 小红 [{语文 90} {数学 100}]}
```

- 序列化：

```golang
package main

import (
	"encoding/json"
	"io/ioutil"
)

type Result struct {
	Name  string  `json:"name"`
	Score float64 `json:"score"`
}

type Student struct {
	Id int `json:"id"`

	Name    string   `json:"name"`
	Results []Result `json:"results"`
}

func main() {
	s := Student{
		Id:   1,
		Name: "小红",
		Results: []Result{
			Result{
				Name:  "语文",
				Score: 90,
			},
			Result{
				Name:  "数学",
				Score: 100,
			},
		},
	}

	dat, _ := json.Marshal(s)
	ioutil.WriteFile("data2.json", dat, 0755)
}
```

当我们运行代码后，打开 `data2.json` 文件，将看到如下内容: 

```json
{
    "id": 1,
    "name": "小红",
    "results": [
        {
            "name": "语文",
            "score": 90
        },
        {
            "name": "数学",
            "score": 100
        }
    ]
}
```
