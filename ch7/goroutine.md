# Goroutine （并发）

并发指的是多个任务被（一个）cpu 轮流切换执行，在 Go 语言里面主要用 goroutine （协程）来实现并发，类似于其他语言中的线程（绿色线程）。

### 语法

```
go f(x, y, z)
```

### 具体例子

首先我们看一个例子：

```golang
package main

import (
	"log"
	"time"
)

func doSomething(id int) {
	log.Printf("before do job:(%d) \n", id)
	time.Sleep(3 * time.Second)
	log.Printf("after do job:(%d) \n", id)
}

func main() {
	doSomething(1)
	doSomething(2)
	doSomething(3)
}
```

输出结果为：
```
2018/03/16 12:13:20 before do job:(1)
2018/03/16 12:13:23 after do job:(1)
2018/03/16 12:13:23 before do job:(2)
2018/03/16 12:13:26 after do job:(2)
2018/03/16 12:13:26 before do job:(3)
2018/03/16 12:13:29 after do job:(3)
```

可以看到执行完结果总共耗时 9 秒，每个任务是阻塞的。

我们可以使用 goroutine 并发执行任务，从而整体加快速度，下面是使用 goroutine 改进的代码：

```golang
package main

import (
	"log"
	"time"
)

func doSomething(id int) {
	log.Printf("before do job:(%d) \n", id)
	time.Sleep(3 * time.Second)
	log.Printf("after do job:(%d) \n", id)
}

func main() {
	go doSomething(1)
	go doSomething(2)
	go doSomething(3)
}
```

当运行代码的时候，会发现没有任何输出。   

这是因为我们的 `main()` 函数其实也是在一个 goroutine 中执行，但是 `main()` 执行完毕后，其他三个 goroutine 还没开始执行，所以就无法看到输出结果。

为了看到输出结果，我们可以使用 `time.Sleep()` 方法让 `main()` 函数延迟结束，例如：

```golang
package main

import (
	"log"
	"time"
)

func doSomething(id int) {
	log.Printf("before do job:(%d) \n", id)
	time.Sleep(3 * time.Second)
	log.Printf("after do job:(%d) \n", id)
}

func main() {
	go doSomething(1)
	go doSomething(2)
	go doSomething(3)
	time.Sleep(4 * time.Second)
}
```

输出结果为：

```
2018/03/16 12:24:23 before do job:(2)
2018/03/16 12:24:23 before do job:(1)
2018/03/16 12:24:23 before do job:(3)
2018/03/16 12:24:26 after do job:(3)
2018/03/16 12:24:26 after do job:(2)
2018/03/16 12:24:26 after do job:(1)
```

可以看到，执行完所有任务从原本的 9 秒下降到 3 秒，大大提高了我们的效率，根据打印输出结果还可以看出：

- 多个 goroutine 的执行是随机。
- 对于 IO 密集型任务特别有效，比如文件，网络读写。

### 使用 `sync.WaitGroup` 实现同步

上面例子中，其实我们还可以使用 `sync.WaitGroup` 来等待所有的 goroutine 结束，从而实现并发的同步，这比使用 `time.Sleep()` 更加优雅，例如：

```golang
package main

import (
	"log"
	"sync"
	"time"
)

func doSomething(id int, wg *sync.WaitGroup) {
	defer wg.Done()

	log.Printf("before do job:(%d) \n", id)
	time.Sleep(3 * time.Second)
	log.Printf("after do job:(%d) \n", id)
}

func main() {
	var wg sync.WaitGroup
	wg.Add(3)

	go doSomething(1, &wg)
	go doSomething(2, &wg)
	go doSomething(3, &wg)

	wg.Wait()
	log.Printf("finish all jobs\n")
}
```

运行代码输出结果为：

```
2018/03/16 13:56:09 before do job:(1)
2018/03/16 13:56:09 before do job:(3)
2018/03/16 13:56:09 before do job:(2)
2018/03/16 13:56:12 after do job:(1)
2018/03/16 13:56:12 after do job:(2)
2018/03/16 13:56:12 after do job:(3)
2018/03/16 13:56:12 finish all jobs
```

### 一个小坑

我们一起来猜猜，下面一段代码运行结果是什么？

```golang
package main

import (
	"fmt"
	"time"
)

func main() {
	for i := 0; i < 3; i++ {
		go func() {
			fmt.Println(i)
		}()
	}

	time.Sleep(1 * time.Second)
}
```

运行代码，输出结果为:

```
3
3
3
```

我们想要的是随机打印 `0,1,2`，但实际输出结果和我们预期不一致，这是原因：

1. 所有 goroutine 代码片段中的 `i` 是同一个变量，待循环结束的时候，它的值为 `3`。
2. `main()` 循环结束后才开始并发执行的新生成的 goroutine。

修复方法：

```golang
package main

import (
	"fmt"
	"time"
)

func main() {
	for i := 0; i < 3; i++ {
		go func(v int) {
			fmt.Println(v)
		}(i)
	}

	time.Sleep(1 * time.Second)
}
```

我们可以通过方法传参的方式，将 `i` 的值拷贝到新的变量 `v` 中，而在每一个 goroutine 都对应了一个属于自己作用域的 `v` 变量，
所以最终打印结果为随机的 `0,1,2`。
