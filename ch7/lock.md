# 锁

在前面我们已经讲了如何使用 channel 在多个 goroutine 之间进行通信，其实对于并发还有一种较为常用通信方式，那就是共享内存。

首先我们来看一个例子：

```
package main

import (
	"log"
	"time"
)

var name string

func main() {
	name = "小明"

	go printName()
	go printName()

	time.Sleep(time.Second)

	name = "小红"

	go printName()
	go printName()

	time.Sleep(time.Second)
}

func printName() {
	log.Println("name is", name)
}
```

运行程序，可以得到类似输出结果：

```
2018/03/23 14:53:28 name is 小明
2018/03/23 14:53:28 name is 小明
2018/03/23 14:53:29 name is 小红
2018/03/23 14:53:29 name is 小红
```

可以看到在两个 goroutine 中我们都可以访问 `name` 这个变量，当修改它后，在不同的 goroutine 中都可以同时获取到最新的值。

这就是一个最简单的通过共享变量（内存）的方式在多个 goroutine 进行通信的方式。

下面再来看一个例子：

```
package main

import (
	"fmt"
	"sync"
)

func main() {
	var (
		wg      sync.WaitGroup
		numbers []int
	)

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(i int) {
			numbers = append(numbers, i)
			wg.Done()
		}(i)
	}

	wg.Wait()

	fmt.Println("The numbers is", numbers)
}
```

多次运行代码，可以得到类似输出:

```
The numbers is [0 1 5 4 7]
The numbers is [0 5 7]
```

可以看到当我们并发对同一个切片进行写操作的时候，会出现数据不一致的问题，这就是一个典型的共享变量的问题。

针对这个问题我们可以使用 Lock（锁）来修复，从而保证数据的一致性，例如：

```
package main

import (
	"fmt"
	"sync"
)

func main() {
	var (
		wg      sync.WaitGroup
		numbers []int

		mux sync.Mutex
	)

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(i int) {
			mux.Lock()
			numbers = append(numbers, i)
			mux.Unlock()

			wg.Done()
		}(i)
	}

	wg.Wait()

	fmt.Println("The numbers is", numbers)
}
```

修改过后，我们再次运行代码，可以看到最后的 numbers 都会包含 0~9 这个10个数字。

`sync.Mutex` 是互斥锁，只有一个信号标量；在 Go 中还有一种读写锁 `sync.RWMutex`，对于我们的共享对象，如果可以分离出读和写两个互斥信号的情况，可以考虑使用它来提高读的并发性能。

例如代码：

```
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
	"time"
)

func main() {
	var (
		mux    sync.Mutex
		state1 = map[string]int{
			"a": 65,
		}
		muxTotal uint64

		rw     sync.RWMutex
		state2 = map[string]int{
			"a": 65,
		}
		rwTotal uint64
	)

	for i := 0; i < 10; i++ {
		go func() {
			for {
				mux.Lock()
				_ = state1["a"]
				mux.Unlock()
				atomic.AddUint64(&muxTotal, 1)
			}
		}()
	}

	for i := 0; i < 10; i++ {
		go func() {
			for {
				rw.RLock()
				_ = state2["a"]
				rw.RUnlock()
				atomic.AddUint64(&rwTotal, 1)
			}
		}()
	}

	time.Sleep(time.Second)

	fmt.Println("sync.Mutex readOps is", muxTotal)
	fmt.Println("sync.RWMutex readOps is", rwTotal)
}
```

运行代码可以得到如下结果：

```
sync.Mutex readOps is 1561870
sync.RWMutex readOps is 15651069
```

可以看到使用 `sync.RWMutex` 的读的并发能力大概是 `sync.Mutex` 的十倍，从而大大提高了其并发能力。

### 总结：

- 我们可以通过共享内存的方式实现多个 goroutine 中的通信。
- 多个 goroutine 对于共享的内存进行写操作的时候，可以使用 `Lock` 来避免数据不一致的情况。
- 对于可以分离为读写操作的共享数据可以考虑使用 `sync.RWMutex` 来提高其读的并发能力。
