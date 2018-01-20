# 命名规范

Go 语言中，任何标识符（变量，常量，函数，自定义类型等）都应该满足以下规律：

- 连续的 [字符](https://golang.org/ref/spec#letter) (unicode_letter | `_` .) 或 [数字](https://golang.org/ref/spec#unicode_digit)("0" … "9") 组成。
- 以字符或下划线开头。
- 不能和 Go 关键字冲突。

### Go 关键字:

```
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

### 举例说明：

```
foo  #合法
foo1 #合法
_foo #合法
变量 #合法
变量1 #合法
_变量 合法

1foo #不合法
1 #不合法
type #不合法
go #不合法

```
