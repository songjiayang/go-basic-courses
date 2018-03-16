# Goroutine （并发）

并发指的是多个任务被（一个）cpu 轮流切换执行，在 Go 语言里面主要用 Goroutine （协程）来实现并发，类似于其他语言中的线程（绿色线程）。

### 语法

```
go function(){}
```

首先我们看一个例子：

```
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

我们可以使用 Goroutine 并发执行任务，从而整体加快速度，下面是使用 Goroutine 改进的代码：

```
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

这是因为我们的 `main()` 函数其实也是在一个 Goroutine 中执行，但是 `main()` 执行完毕后，其他三个 Goroutine 还没开始执行，所以就无法看到输出结果。

为了看到输出结果，我们可以使用 `time.Sleep()` 方法让 `main()` 函数延迟结束，例如：

```
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

- 多个 Goroutine 的执行是随机。
- 对于 IO 密集型任务特别有效，比如文件，网络读写。

