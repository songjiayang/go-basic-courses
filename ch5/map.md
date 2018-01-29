# Map

在 Go 语言里面，map 一种无序的键值对, 它是数据结构 hash 表的一种实现方式，类似 Python 中的字典。

### 语法

使用关键字 map 来声明形如：

```
map[KeyType]ValueType
```

注意点：

- 必须指定 key, value 的类型，插入的纪录类型必须匹配。
- key 具有唯一性，插入纪录的 key 不能重复。
- KeyType 可以为基础数据类型（例如 bool, 数字类型，字符串）, 不能为数组，切片，map，它的取值必须是能够使用 `==` 进行比较。
- ValueType 可以为任意类型。
- 无序性。
- 线程不安全, 一个 goroutine 在对 map 进行写的时候，另外的 goroutine 不能进行读和写操作，Go 1.6 版本以后会抛出 runtime 错误信息。

### 声明和初始化

- 使用 var 声明

```golang
var cMap map[string]int  // 只定义, 此时 cMap 为 nil
fmt.Println(cMap == nil)
cMap["北京"] = 1  // 报错，因为 cMap 为 nil
```

- 使用 `make`

```golang
cMap := make(map[string]int)
cMap["北京"] = 1

// 指定初始容量
cMap = make(map[string]int, 100)
cMap["北京"] = 1
```

说明：在使用 make 初始化 map 的时候，可以指定初始容量，这在能预估 map key 数量的情况下，减少动态分配的次数，从而提升性能。

- 简短声明方式

```
cMap := map[string]int{"北京": 1}
```

### map 基本操作

```
cMap := map[string]int{}

cMap["北京"] = 1 //写

code := cMap["北京"] // 读
fmt.Println(code)

code = cMap["广州"]  // 读不存在 key
fmt.Println(code)

code, ok = cMap["广州"]  // 检查 key 是否存在
if ok {
  fmt.Println(code)  
} else {
  fmt.Println("key not exist")  
}

delete(cMap, "北京") // 删除 key
fmt.Println("北京")
```

### 循环和无序性

```
cMap := map[string]int{"北京": 1, "上海": 2, "广州": 3, "深圳": 4}

for city, code := range cMap {
  fmt.Printf("%s:%d", city, code)
  fmt.Println()
}
```

### 线程不安全

```
cMap := make(map[string]int)

var wg sync.WaitGroup
wg.Add(2)

go func() {
	cMap["北京"] = 1
	wg.Done()
}()

go func() {
	cMap["上海"] = 2
	wg.Done()
}()

wg.Wait()
```

在 Go 1.6 之后的版本，多次运行此段代码，你将遇到这样的错误信息：

```
fatal error: concurrent map writes

goroutine x [running]:
runtime.throw(0x10c64b6, 0x15)
.....
```

解决之道：

- 对读写操作加锁
- 使用 security map, 例如 `sync.map`

### map 嵌套

```
provinces := make(map[string]map[string]int)

provinces["北京"] = map[string]int{
  "东城区": 1,
  "西城区": 2,
  "朝阳区": 3,
  "海淀区": 4,
}

fmt.Println(provinces["北京"])
```
