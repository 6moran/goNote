### 协程：

**并发：**多线程程序在单核上运行，就是并发。

**并行：**多线程程序在多核上运行，就是并行。

GO主线程（有程序员直接称为线程/也可以理解为进程）：一个GO线程上，可以起多个协程，**协程是轻量级的线程**。



**1.协程的特点：**

- 有独立的栈空间
- 共享程序堆空间
- 调度由用户控制
- 协程是轻量级的线程

**示例：**

```go
func test1() {
    for i := 1; i <= 10; i++ {
       fmt.Println("test() hello,world " + strconv.Itoa(i))
       time.Sleep(time.Second)
    }
}
func main() {
	go test1()
	for i := 1; i <= 10; i++ {
		fmt.Println("test() hello,golang " + strconv.Itoa(i))
		time.Sleep(time.Second)
	}
}
```

![image-20250922202607870](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250922202607870.png)

主线程结束了，协程即使没结束程序也会结束。



**线程小结：**

1.主线程是一个物理线程，直接作用在cpu上的。是重量级的，非常耗费cpu资源。

2.协程是从主线程开启的，是轻量级的线程，是逻辑态。对资源消耗相对小。

3.Golang的协程机制是重要的特点，可以轻松的开启上万个协程。其他语言的并发机制一般是基于线程的，开启过多的线程，资源耗费大，这里就凸显Golang在并发上的优势了。



**MPG模式基本介绍：**

1.M：操作系统的主线程（物理线程）。

2.P：协程执行需要的上下文。

3.G：协程。



资源竞争的解决方案(解决goroutine的通讯问题)：

### 锁:

1. 互斥锁:

```go
var (
    myMap = make(map[int]int, 10)
    //声明一个全局的互斥锁
    //sync是一个包：synchorniized 同步
    //Mutex是互斥
    lock sync.Mutex
)

//加锁访问map资源，避免资源竞争
lock.Lock()
myMap[n] = res
lock.Unlock()
//操作系统也要加锁，不然会资源竞争
lock.Lock()
for i, v := range myMap {
    fmt.Printf("map[%d]=%d\n", i, v)
}
lock.Unlock()
```

读写互斥锁：

![image-20250929220007251](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250929220007251.png)

读写锁分为两种：读锁和写锁。当一个 goroutine 获取到读锁之后，其他的 goroutine 如果是获取读锁会继续获得锁，如果是获取写锁就会等待；而当一个 goroutine 获取写锁之后，其他的 goroutine 无论是获取读锁还是写锁都会等待。

### 管道：

**channel管道：**

`channle`本质就是一个队列，FIFO（先进先出）

线程安全，多goroutine访问时，不需要加锁，就是说channel本身就是线程安全的

（多个协程操作同一个管道时，不会发生资源竞争）

​	1. 管道的初始化：

```go
//管道
intChan := make(chan int, 3)
```

​	2. 管道的写入和取出：

```go
//管道写入数据时，不能超过其容量，管道不会动态扩容
intChan <- 10
intChan <- 23
//取出，取出时管道长度会会变化
//当管道中数据已经全部取出时，再取就会报错deadlock
num := <-intChan
```

​	3. 注意事项：

- channel中只能存放指定的数据类型。
- channel数据放满后就不能再放入了。
- 如果从channel取出数据后，可以继续放入。
- 在没有使用协程的情况下，如果channel的数据取完了，再取就会报deadlock。
- 空接口管道在取出进行使用时要进行类型断言。

​	4. 管道的关闭与遍历：

```go
//当通道未关闭且有数据时，ok为true
//当通道已关闭且无数据可接收时，ok为false
//当通道已经关闭但还有剩余数据可接收时，ok 的值为 true。
//当通道未关闭且当前没有数据可接收时，接收操作会阻塞，程序会暂停在接收语句处，直到通道中有新数据发送过来，或者通道被关闭。
<-boolChan
//读取阻塞直到关闭
```

管道关闭后，只能读取不能写入。

channel支持for-range遍历，不能使用普通for循环。

```go
close(intChan2)
//在遍历时，如果管道没关闭，会报deadlock的错误
for v := range intChan2 {
    fmt.Println(v)
}
```

​	5. 双向通道：

```go
<- chan int // 只接收通道，只能接收不能发送
chan <- int // 只发送通道，只能发送不能接收
//它的类型仍是chan int，只读只写算是它的一个属性，双向通道仍能传给它
```

![image-20250924201550846](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250924201550846.png)

*nil为没有空间，即只进行了声明。*

**注意：**对已经关闭的通道再执行 close 也会引发 panic。



**sync：**

`sync.WaitGroup`控制协程和主程序的执行，协调多个 goroutine 同步。

**运用实例：**

```go
package main

import (
    "fmt"
    "sync"
)

var wg sync.WaitGroup

// 写入
func writeData(intChan chan int) {
    for i := 1; i <= 2000; i++ {
       intChan <- i
    }
    close(intChan)
}

func readChan(intChan chan int, resChan chan int) {
    defer wg.Done()
    for {
       v, ok := <-intChan
       if !ok {
          break
       }
       sum := 0
       for i := 1; i <= v; i++ {
          sum += i
       }
       resChan <- sum
    }
}

func main() {
    numChan := make(chan int, 2000)
    resChan := make(chan int, 2000)

    go writeData(numChan)
    for i := 0; i < 8; i++ {
       go readChan(numChan, resChan)
       wg.Add(1)
    }
    wg.Wait()
    close(resChan)
    for v := range resChan {
       fmt.Println(v)
    }
    
```

写入1-2000到`intChan`管道，将其连加写入到`resChan`管道，最后打印。



sync包中提供了一个针对只执行一次场景的解决方案`sync.Once`，`sync.Once`只有一个`Do`方法，

`func (o *Once) Do(f func())`

运用场景:

```go
var loadIconsOnce sync.Once
loadIconsOnce.Do(loadIcons)	//只执行一次loadIcons函数
```



Go 语言中内置的 map 不是并发安全的，我们不能在多个 goroutine 中并发对内置的 map 进行读写操作，否则会存在数据竞争问题。

Go语言的`sync`包中提供了一个开箱即用的并发安全版 map，开箱即用表示其不用像内置的 map 一样使用 make 函数初始化就能直接使用。

![image-20250929093215647](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250929093215647.png)







**channel注意事项：**

select

```go
//当不知道/不确定什么时候该关闭管道而又想打印时
//可以用select方式
//这里要退出for循环可以label:但是不建议，可以将其写为一个函数来搞，最后直接return
for {
    select {
    //注意：这里，如果intChan一直没有关闭，不会一直阻塞而deadlock
    //会自动执行下一个case匹配
    case v := <-intChan:
       fmt.Printf("从intChan中读取的数据%d\n", v)
       time.Sleep(time.Second)
    case v := <-stringChan:
       fmt.Printf("从stringChan中读取的数据%s\n", v)
       time.Sleep(time.Second)
    default:
       fmt.Println("没有数据可以取出")
       time.Sleep(time.Second)
       return
    }
}
```



如果一个协程出现了panic，这时不想让它影响整个程序，就可以在该协程将panic捕获，defer recover。



