# 原子操作

当我们想在多个 goroutine 实现累加或者递减操作的时候，不仅可以使用前面学到的加锁的操作还可以使用 `atomic`。


### 使用锁实现

```
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var (
		mux   sync.Mutex
		total = 0
	)

	for i := 0; i < 10; i++ {
		go func() {
			for {
				mux.Lock()
				total += 1
				mux.Unlock()
				time.Sleep(time.Millisecond)
			}
		}()
	}

	time.Sleep(time.Second)
	fmt.Println("The total number is", total)
}
```

运行代码可以得到类似输出：

```
The total number is 7770
```

### 使用 atomic 实现

```
package main

import (
	"fmt"
	"sync/atomic"
	"time"
)

func main() {
	var total int64

	for i := 0; i < 10; i++ {
		go func() {
			for {
				atomic.AddInt64(&total, 1)
				time.Sleep(time.Millisecond)
			}
		}()
	}

	time.Sleep(time.Second)
	fmt.Println("The total number is", total)
}
```

运行代码可以得到类似输出：

```
The total number is 7701
```

### 结论

- `atomic` 和锁的方式其性能没有太大区别。
- `atomic` 写法相较于使用锁，更简单。
