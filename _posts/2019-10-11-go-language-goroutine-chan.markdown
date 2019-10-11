---
layout: post
title: "Go语言-Goroutine和通道"
date: 2019-10-11
tag: Go Goroutine channel
---
- [goroutine](#org9a97dab)
  - [并发和并行](#org82dec61)
    - [模拟阻塞执行](#org902a101)
    - [使用 go关键字](#orgd485614)
    - [阻止程序直接退出](#org35df47e)
  - [Goroutine](#org7318b8e)
- [通道 chan](#org51087eb)
  - [使用通道进行通信](#orgc30933b)
  - [使用缓冲通道](#orgd2631bb)
  - [流程控制](#orgd7cd7b2)
    - [通道接收一条消息就返回](#org9d590f5)
    - [创建不断监听通道中消息的监听器](#orga484b15)
  - [将通道作为函数参数](#org3a9fe4e)
    - [指定通道访问权限](#org18a2deb)
  - [select 语句](#org9b52b5c)
    - [结合使用 select 语句和通道](#org4c4de33)
    - [给 select 语句指明超时时间](#org1e8156c)
  - [退出通道](#org3d73803)
  - [小练习](#org512602f)


<a id="org9a97dab"></a>

# goroutine

-   顺序执行 ：一个任务完成之后再执行下一个
-   并行： 不必等到一个操作执行完毕后再执行下一个


<a id="org82dec61"></a>

## 并发和并行

并发就是同时处理很多事情，而并行就是同时做很多事情。

-   并行：指同一时刻同时进行，进程并行需要多个处理器支持。
-   并发：指一段时间内的多个进程轮流切换使CPU。


<a id="org902a101"></a>

### 模拟阻塞执行

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// 模拟阻塞的代码
package main

import (
        "fmt"
        "time"
)

func main () {
        slowFunc()
        fmt.Println("I'm not shown until slowFunc() completes.")
}

// 
func slowFunc () {
        fmt.Println("Sleepper() start")
        time.Sleep(time.Second)
        fmt.Println("Sleepper() finished")
}
```

goroutine 使用： 只需要让Goroutine执行的函数或方法前加上关键字go即可


<a id="orgd485614"></a>

### 使用 go关键字

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// 使用goroutine
package main

import (
        "fmt"
        "time"
)

func main () {
        go slowFunc()
        fmt.Println("I'm not shown straightaway!")
}

// 
func slowFunc () {
        fmt.Println("Sleeper() started")
        time.Sleep(time.Second)
        fmt.Println("Sleeper() finished")
}


```

程序输出一行就直接退出了，因为goroutine 立即返回，程序直接执行后面的代码，然后退出，如果没有其他因素阻止，程序将在 goroutine 返回前退出。


<a id="org35df47e"></a>

### 阻止程序直接退出

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// 使用goroutine
import (
        "fmt"
        "time"
)

func main () {
        go slowFunc()
        fmt.Println("I'm not shown straightaway!")
        time.Sleep(time.Second * 2)
}

// 
func slowFunc () {
        fmt.Println("Sleeper() started")
        time.Sleep(time.Second)
        fmt.Println("Sleeper() finished")
}
```


<a id="org7318b8e"></a>

## Goroutine

-   go 在幕后使用线程来管理并发，但Goroutine 让程序员无需直接管理线程。
-   创建一个Goroutine 只需要占用几KB内存，因此即便创建数千个 Goroutine 也不会耗尽内存。
-   另外创建和销毁Goroutine 的效率也非常的高。

-   并发地执行代码意味着程序可能更快的执行完毕，并在数据就绪后就返回，而无需等待程序的其他部分结束。
-   阻塞式的代码让程序暂停执行。
-   使用并法编程的情景包括从磁盘文件读取数据，从网络读写数据以及从数据库读写数据。


<a id="org51087eb"></a>

# 通道 chan

-   通道和 Goroutine 一起提供了一个受控的环境，通道让 Goroutine 能够互相通信，用于开发并发软件。

如果说 Goroutine 是一种支持并发编程的方式，那么通道就是一种与 Goroutine 通信的方式。 通道让数据能够进入和离开 Goroutine， 可方便 Goroutine 之间的通信。

-   不要通过共享内存来通信，而通过通信来共享内存
-   通道只能有一种数据类型，可以创建任意类型的通道，因此可以使用结构体来存储复杂的数据结构


<a id="orgc30933b"></a>

## 使用通道进行通信

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// 使用通道进行通信

import (
        "fmt"
        "time"
)

func main () {
        c := make(chan string)
        go slowFunc(c)

        msg := <-c
        fmt.Println(msg)
}

// 
func slowFunc (c chan string) {
        time.Sleep(time.Second * 2)
        c <- "slowFunc() finished"
}
```


<a id="orgd2631bb"></a>

## 使用缓冲通道

-   缓冲通道只能存储指定数量的消息

如果向它发送更多的消息，将导致错误

-   close 用来关闭通道，禁止再向通道发送消息

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// 
import (
        "fmt"
        "time"
)

func main () {
        messages := make(chan string, 2)
        messages <- "hello"
        messages <- "world"

        close(messages)
        fmt.Println("Pushed two messages onto Channel with no receivers")
        time.Sleep(time.Second)
        reveiver(messages)
}

// 
func reveiver (c chan string) {
        for msg := range c {
                fmt.Println(msg)
        }
}
```


<a id="orgd7cd7b2"></a>

## 流程控制

-   给通道指定消息接收者是一个阻塞操作，因为它将阻止函数返回，直到收到一条消息为止。
-   从通道接收并打印消息的程序需要阻塞，以免终止。


<a id="org9d590f5"></a>

### 通道接收一条消息就返回

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// 通道接收一条消息就返回

import (
        "fmt"
        "time"
)

func main () {
        messages := make(chan string)
        go pinger(messages)
        msg := <- messages
        fmt.Println(msg)
}

// 
func pinger (c chan string) {
        t := time.NewTicker(1 * time.Second)
        for {
                c <- "ping"
                <- t.C
        }
}
```


<a id="orga484b15"></a>

### 创建不断监听通道中消息的监听器

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// 不间断的指定接收操作
import (
        "fmt"
        "time"
)

func main () {
        messages :=  make(chan string)
        go pinger(messages)
        for i := 0 ; i < 5 ; i ++ {
                msg := <- messages
                fmt.Println(msg)
        }
}

// 
func pinger (c chan string) {
        t := time.NewTicker(1 * time.Second)
        for {
                c <- "ping"
                <- t.C
        }
}

```


<a id="org3a9fe4e"></a>

## 将通道作为函数参数

-   可以进一步指定在函数中如何使用传入的参数，可在传递通道时指定为只读，只写或读写的。


<a id="org18a2deb"></a>

### 指定通道访问权限

```go
func channelReader(messages <-chan string){
        mes := messages
        fmt.Println(msg)
}

// 通道在函数内只写的
func channelWriter (messages chan<- string) {
        messages <- "messages"
}

// 没有指定箭头，表示通道可读可写
func channelReaderWriter (messages chan string) {
        msg := <- messages
        fmt.Println(msg)
        messages <- "helloworld"
}
```


<a id="org9b52b5c"></a>

## select 语句


<a id="org4c4de33"></a>

### 结合使用 select 语句和通道

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// select 语句使用

import (
        "fmt"
        "time"
)

func main () {
        channel1 := make(chan string)
        channel2 := make(chan string)

        go ping1(channel1)
        go ping2(channel2)

        select {
        case msg1 := <- channel1:
                fmt.Println("reveived", msg1)
        case msg2 := <- channel2:
                fmt.Println("reveived", msg2)
        }
}

// 
func ping1 (c chan string) {
        time.Sleep(time.Second)
        c <- "ping on channel1"
}

// 
func ping2 (c chan string) {
        time.Sleep(time.Second * 2)
        c <- "ping on channel2"
}
```

-   哪条消息最先到达，将执行哪条case 语句
-   即根据最先收到的消息采取相应的措施


<a id="org1e8156c"></a>

### 给 select 语句指明超时时间

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// 给select 语句指定超时时间

import (
        "fmt"
        "time"
)

func main () {
        channel1 := make(chan string)
        channel2 := make(chan string)

        go ping1(channel1)
        go ping2(channel2)

        select {
        case msg1 := <- channel1:
                fmt.Println("reveived", msg1)
        case msg2 := <- channel2:
                fmt.Println("reveived", msg2)
        case <- time.After(500 * time.Millisecond):
                fmt.Println("no messages receiverd. giving up")
        }
}

// 
func ping1 (c chan string) {
        time.Sleep(time.Second)
        c <- "ping on channel1"
}

// 
func ping2 (c chan string) {
        time.Sleep(time.Second * 2)
        c <- "ping on channel2"
}
```


<a id="org3d73803"></a>

## 退出通道

在已知需要停止的时间的情况下，使用超时时间是不错的选择，在不确定 select 语句该在何时返回， 因此不能使用定时器。 在这种情况下可以使用退出通道。

-   通过在 select 语句中添加一个退出通道，可向退出通道发送消息来结束该语句，从而停止阻塞。
-   关闭缓冲意味着不能再向它发送消息。缓冲的消息会被保留，可供接收者读取。

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// 使用退出通道

import (
        "time"
        "fmt"
)

func main () {
        messages :=  make(chan string)
        stop := make(chan bool)

        go sender(messages)
        go func() {
                time.Sleep(time.Second * 2)
                fmt.Println("time is up!")
                stop <- true
        }()

        for {
                select {
                case <- stop :
                        return
                case msg := <- messages:
                        fmt.Println(msg)
                }
        }
}

// 
func sender (c chan string) {
        t := time.NewTicker(time.Second)
        for {
                c <- "I'm sending a messages"
                <- t.C
        }
}
```


<a id="org512602f"></a>

## 小练习

接收10 条消息后退出程序

```go
// Filename: goroutine_chan.org[*Org Src goroutine_chan.org[ go ]*]
// coding:utf-8
// 通道接收后退出

import (
        "fmt"
        "time"
)

func main () {
        messages := make(chan string)

        go sender(messages)

        for i := 0; i < 10; i++ {
                msg :=  <- messages
                fmt.Println(msg)
        }
}

// send messages
func sender (c chan string) {
        t := time.NewTicker(time.Second)
        for {
                c <- "This is a messages."
                <- t.C
        }
}
```
