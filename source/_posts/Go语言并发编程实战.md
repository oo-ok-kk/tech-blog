---
title: Go 语言并发编程实战
date: 2024-06-01
tags:
  - Go
  - 并发
  -  Goroutine
cover: https://images.unsplash.com/photo-1550439062-609e1531270e?w=800
---

Go 语言以其简洁的并发模型著称，goroutine 和 channel 让并发编程变得优雅而高效。本文通过实战案例深入讲解 Go 的并发编程。

## 一、Go 并发基础

### Goroutine

Goroutine 是 Go 的轻量级线程，由 Go 运行时管理：

```go
// 启动一个 goroutine
go func() {
    fmt.Println("Hello from goroutine!")
}()

// 带参数的 goroutine
go func(msg string) {
    fmt.Println(msg)
}("Hello")
```

### Channel

Channel 是 goroutine 之间的通信机制：

```go
// 创建 channel
ch := make(chan int)

// 发送数据
ch <- 10

// 接收数据
num := <-ch

// 关闭 channel
close(ch)
```

## 二、并发模式

### 1. 管道（Pipeline）

```go
func main() {
    // 生成数据
    naturals := make(chan int)
    
    // 平方计算
    squares := make(chan int)
    
    // 启动三个 goroutine
    go func() {
        for x := 0; x < 10; x++ {
            naturals <- x
        }
        close(naturals)
    }()
    
    go func() {
        for x := range naturals {
            squares <- x * x
        }
        close(squares)
    }()
    
    // 打印结果
    for x := range squares {
        fmt.Println(x)
    }
}
```

### 2. 扇入扇出（Fan-out, Fan-in）

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        fmt.Printf("worker %d processing job %d\n", id, job)
        results <- job * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)
    
    // 启动3个 worker
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }
    
    // 发送9个任务
    for j := 1; j <= 9; j++ {
        jobs <- j
    }
    close(jobs)
    
    // 收集结果
    for a := 1; a <= 9; a++ {
        <-results
    }
}
```

### 3. 等待组（sync.WaitGroup）

```go
func main() {
    var wg sync.WaitGroup
    
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            fmt.Println("Goroutine", n, "finished")
        }(i)
    }
    
    wg.Wait()
    fmt.Println("All goroutines finished")
}
```

### 4. 互斥锁（sync.Mutex）

```go
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}
```

### 5. 读写锁（sync.RWMutex）

```go
type Cache struct {
    mu    sync.RWMutex
    data  map[string]string
}

func (c *Cache) Get(key string) string {
    c.mu.RLock()
    defer c.mu.RUnlock()
    return c.data[key]
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = value
}
```

## 三、Context 上下文

Context 用于在 goroutine 之间传递取消信号和截止时间：

```go
func longRunningTask(ctx context.Context) error {
    select {
    case <-time.After(2 * time.Second):
        return nil
    case <-ctx.Done():
        return ctx.Err()
    }
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()
    
    err := longRunningTask(ctx)
    if err != nil {
        fmt.Println("Task cancelled:", err)
    }
}
```

## 四、原子操作

对于简单的计数器，使用 sync/atomic 避免锁：

```go
import "sync/atomic"

var counter int64

func increment() {
    atomic.AddInt64(&counter, 1)
}

func get() int64 {
    return atomic.LoadInt64(&counter)
}
```

## 五、Select 多路复用

```go
func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "one"
    }()
    
    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "two"
    }()
    
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println("Received:", msg1)
        case msg2 := <-ch2:
            fmt.Println("Received:", msg2)
        }
    }
}
```

### 超时处理

```go
select {
case result := <-ch:
    fmt.Println("Result:", result)
case <-time.After(5 * time.Second):
    fmt.Println("Timeout!")
}
```

### 退出信号

```go
select {
case <-ch:
    // 处理数据
case <-done:
    // 退出
    return
}
```

## 六、并发安全 Map

Go 1.9+ 提供了 sync.Map：

```go
var m sync.Map

// 存储
m.Store("key", "value")

// 读取
value, ok := m.Load("key")

// 删除
m.Delete("key")

// 遍历
m.Range(func(k, v interface{}) bool {
    fmt.Println(k, v)
    return true
})
```

## 七、实战：并发爬虫

```go
func crawl(url string, wg *sync.WaitGroup) {
    defer wg.Done()
    
    resp, err := http.Get(url)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    defer resp.Body.Close()
    
    fmt.Println("Crawled:", url, "Status:", resp.Status)
}

func main() {
    urls := []string{
        "https://golang.org",
        "https://github.com",
        "https://stackoverflow.com",
    }
    
    var wg sync.WaitGroup
    for _, url := range urls {
        wg.Add(1)
        go crawl(url, &wg)
    }
    wg.Wait()
    fmt.Println("Done!")
}
```

## 八、总结

| 特性 | 说明 |
|------|------|
| Goroutine | 轻量级线程 |
| Channel | 通信机制 |
| sync.WaitGroup | 等待组 |
| sync.Mutex | 互斥锁 |
| sync.RWMutex | 读写锁 |
| context | 取消信号 |
| atomic | 原子操作 |

Go 的并发模型简单而强大，充分利用了现代多核CPU的性能。
