#### 通知子goroutine退出的方法

- 全局变量方法

```
var wg sync.WaitGroup
var exit bool

// 全局变量方式存在的问题：
// 1. 使用全局变量在跨包调用时不容易统一
// 2. 如果worker中再启动goroutine，就不太好控制了。

func worker() {
	for {
		fmt.Println("worker")
		time.Sleep(time.Second)
		if exit {
			break
		}
	}
	wg.Done()
}

func main() {
	wg.Add(1)
	go worker()
	time.Sleep(time.Second * 3) // sleep3秒以免程序过快退出
	exit = true                 // 修改全局变量实现子goroutine的退出
	wg.Wait()
	fmt.Println("over")
}
```

- 通道方法

  ```
  var wg sync.WaitGroup
  
  // 管道方式存在的问题：
  // 1. 使用全局变量在跨包调用时不容易实现规范和统一，需要维护一个共用的channel
  
  func worker(exitChan chan struct{}) {
  LOOP:
  	for {
  		fmt.Println("worker")
  		time.Sleep(time.Second)
  		select {
  		case <-exitChan: // 等待接收上级通知
  			break LOOP
  		default:
  		}
  	}
  	wg.Done()
  }
  
  func main() {
  	var exitChan = make(chan struct{})
  	wg.Add(1)
  	go worker(exitChan)
  	time.Sleep(time.Second * 3) // sleep3秒以免程序过快退出
  	exitChan <- struct{}{}      // 给子goroutine发送退出信号
  	close(exitChan)
  	wg.Wait()
  	fmt.Println("over")
  }
  ```

  - 官方推荐方法 context. 专门用来简化 对于处理单个请求的多个 goroutine 之间与请求域的数据、取消信号、截止时间等相关操作，这些操作可能涉及多个 API 调用 

```
var wg sync.WaitGroup

func worker(ctx context.Context) {
LOOP:
	for {
		fmt.Println("worker")
		time.Sleep(time.Second)
		select {
		case <-ctx.Done(): // 等待上级通知
			break LOOP
		default:
		}
	}
	wg.Done()
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(1)
	go worker(ctx)
	time.Sleep(time.Second * 3)
	cancel() // 通知子goroutine结束
	//当子goroutine又开启goroutine时,只需传入ctx
	wg.Wait()
	fmt.Println("over")
}
```

