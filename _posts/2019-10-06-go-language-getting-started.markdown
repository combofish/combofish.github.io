---
layout: post
title: "Go language Getting Started"
date: 2019-10-06
tag: Go 
---
- [基础知识](#org461363b)
  - [hello world](#orgde37139)
  - [无限循环](#org15f1aae)
  - [slice](#orgd120523)
  - [echo 将参数输出](#org36aa3e8)
    - [echo1.go](#org8302d4c)
    - [echo2.go](#orgfa425a0)
  - [几种声明字符串变量的方式](#org2fd7e20)
  - [从文件中统计重复行](#org88144ab)
  - [gif 动画](#org58dddfe)
  - [fetch 获取](#orgecfd5b5)
    - [fetch 从URL获取的内容](#orgf142caa)
    - [fetchall 从多个url获取的内容](#org78900d0)
    - [fetchall<sub>2</sub> 从文件中的url获取的内容](#org8b788d4)
  - [一个 web 服务器](#org970e165)
    - [server1.go 返回请求路径](#orgb819268)
    - [server2.go 一个迷你回声和计数服务器](#orgb98c0f3)
    - [server3.go 处理程序回显HTTP 请求](#orgc5fee37)
    - [server4.go 同过网页显示 lissajous 图形](#org10d1fde)
  - [控制流](#org845f71a)
  - [实现 Stringer interface](#org8d42312)
  - [bytecounter](#org2bdb268)
  - [geometry](#org85e361b)
  - [fib 函数](#org60aa654)
  - [数据类型](#orgc9d03c2)
    - [basic type](#org01d8c9f)
    - [aggregate type](#org7e9769b)
    - [reference type](#org88965c4)
    - [interface type](#orga12f9c4)


<a id="org461363b"></a>

# 基础知识

作者： Robert Griesemer, Rob Pike, Ken Thompson.

-   基础架构语言

特被适合构建基础设施类软件如网络服务器，以及程序员使用的工具和系统等。通用语言。

-   CSP Communicating Sequential Process, 通信顺序进程。


<a id="orgde37139"></a>

## hello world

```go
package main

import "fmt"

func main() {
        fmt.Println('hello world!')
}
```

```sh
go run helloworld.go
go build helloworld.go
./helloworld
```

名为main的包比较特殊，他用来定义一个独立的可执行程序，而不是库。在main 包中，函数main 总是程序开始执行的地方。


<a id="org15f1aae"></a>

## 无限循环

```go
for {

}
```


<a id="orgd120523"></a>

## slice

索引使用半开区间，即包含第一个索引，不包含最后一个索引。 slice s[m:n] ,其中 0<<m <<n << len(s) 包含 n - m 个元素。


<a id="org36aa3e8"></a>

## echo 将参数输出


<a id="org8302d4c"></a>

### echo1.go

```go
package main

import (
        "fmt"
        "os"
)

func main(){
        var s, sep string
        for  i := 1; i< len(os.Args); i++ {
                s += sep + os.Args[i]
                sep = " "
        }
        fmt.Println(s)
}

```

如果变量没有明确的初始化，它将隐式地初始化这个类型的空值。例如，对于数字初始化是0, 对于字符串是空字符串。 += 是一个赋值操作符


<a id="orgfa425a0"></a>

### echo2.go

```go
package main

import (
        "fmt"
        "os"
)

func main () {
        s, sep := "", ""
        for _, arg := range os.Args[1:] {
                s += sep + arg
                sep = " "
        }
        fmt.Println(s)
}
```

空标识符，可以用在任何语法需要变量名但是程序逻辑不需要的地方。例如丢弃每次迭代产生的无用的索引。


<a id="org2fd7e20"></a>

## 几种声明字符串变量的方式

```go
s := "" //短变量的声明方式更加简洁
var s string
var s = ""
var s string = ""
```

-   map 存储一个键值对集合。map里的键的迭代顺序不是固定的。通常是随机的，每次运行都不一致。 这是有意设计的，以防止程序依赖某种特定的序列。

-   函数以及其他包级别的实体可以以任意次序声明，比如函数调用可以出现在声明之前。


<a id="org88144ab"></a>

## 从文件中统计重复行

-   dup3
    
    ```go
    // 统计文件中不重复的行的个数
    package main
    
    import (
            "fmt"
            "io/ioutil"
            "os"
            "strings"
    )
    
    func main() {
            counts := make(map[string]int)
            for _, filename := range os.Args[1:] {
                    data, err := ioutil.ReadFile(filename)
                    fmt.Println(filename)
    
                    if err != nil {
                            fmt.Fprintf(os.Stderr, "dup3: %v\n", err)
                            continue
                    }
                    for _, line := range strings.Split(string(data), "\n"){
                            counts[line]++
                    }
            }
            for line, n := range counts {
                    if n > 1 {
                            fmt.Printf("%d\t%s\n", n, line)
                    }
            }
    
    }
    ```


<a id="org58dddfe"></a>

## gif 动画

-   lissajous 产生随机里萨如图形的gif
    
    ```go
    // lissajous 产生随机里萨如图形的GIF 动画
    package main
    
    import (
            "image"
            "image/color"
            "image/gif"
            "io"
            "log"
            "math"
            "math/rand"
            "net/http"
            "os"
            "time"
    )
    
    var palette = []color.Color{color.White, color.Black}
    
    const (
            whiteIndex = 0
            blackIndex = 1
    )
    
    func main() {
            rand.Seed(time.Now().UTC().UnixNano())
            if len(os.Args) > 1 && os.Args[1] == "web" {
                    handler := func(w http.ResponseWriter, r *http.Request) {
                            lissajous(w)
                    }
                    http.HandleFunc("/", handler)
                    log.Fatal(http.ListenAndServe("localhost:8000", nil))
                    return
            }
            lissajous(os.Stdout)
    }
    
    func lissajous(out io.Writer) {
            const (
                    cycles  = 5
                    res     = 0.001
                    size    = 100
                    nframes = 64
                    delay   = 8
            )
            freq := rand.Float64() * 3.0
            anim := gif.GIF{LoopCount: nframes}
            phase := 0.0
            for i := 0; i < nframes; i++ {
                    rect := image.Rect(0, 0, 2*size+1, 2*size+1)
                    img := image.NewPaletted(rect, palette)
                    for t := 0.0; t < cycles*2*math.Pi; t += res {
                            x := math.Sin(t)
                            y := math.Sin(t*freq + phase)
                            img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5), blackIndex)
                    }
                    phase += 0.1
                    anim.Delay = append(anim.Delay, delay)
                    anim.Image = append(anim.Image, img)
            }
            gif.EncodeAll(out, &anim)
    }
    ```


<a id="orgecfd5b5"></a>

## fetch 获取


<a id="orgf142caa"></a>

### fetch 从URL获取的内容

```go
// fetch 输出从URL 获取到的内容。
package main

import (
        "fmt"
        "io/ioutil"
        "net/http"
        "os"
)

func main() {
        for _, url := range os.Args[1:] {
                resp, err := http.Get(url)
                if err != nil {
                        fmt.Fprintf(os.Stderr, "fetch: %v \n", err)
                        os.Exit(1)
                }
                b, err := ioutil.ReadAll(resp.Body)
                resp.Body.Close()
                if err != nil {
                        fmt.Fprintf(os.Stderr, "fetch: reading %s : %v\n", url, err)
                        os.Exit(1)
                }
                fmt.Printf("%s", b)
        }
}
```


<a id="org78900d0"></a>

### fetchall 从多个url获取的内容

```go
//fetchall 并法获取URL 并报告他们的时间和大小
package main

import (
        "fmt"
        "io"
        "io/ioutil"
        "net/http"
        "os"
        "time"
)

func main(){
        start := time.Now()
        ch := make(chan string)
        for _, url := range os.Args[1:] {
                go fetch(url, ch)
        }
        for range os.Args[1:] {
                fmt.Println(<-ch)
        }
        fmt.Printf("%.3fs elapsed\n", time.Since(start).Seconds())
}

func fetch(url string, ch chan <- string) {
        start := time.Now()
        resp, err := http.Get(url)
        if err != nil {
                ch <- fmt.Sprint(err)
                return
        }

        nbytes, err := io.Copy(ioutil.Discard, resp.Body)
        resp.Body.Close()
        if err != nil {
                ch <- fmt.Sprintf("While reading %s: %v", url, err)
                return
        }

        secs := time.Since(start).Seconds()
        ch <- fmt.Sprintf("%.3fs %7d %s", secs, nbytes, url)
}
```


<a id="org8b788d4"></a>

### fetchall<sub>2</sub> 从文件中的url获取的内容

```go
//fetchall 并法获取URL 并报告他们的时间和大小
package main

import (
        "fmt"
        "io"
        "io/ioutil"
        "net/http"
        "os"
        "strings"
        "time"
)

func main() {
        start := time.Now()
        ch := make(chan string)
        filename := os.Args[1]
        data, err := ioutil.ReadFile(filename)
        if err != nil {
                fmt.Fprintf(os.Stderr, "fetchall_2:%v \n", err)
        }

        l_ := strings.Split(string(data), "\n")
        length_ := len(l_)

        for _, line := range l_ {
                go fetch(line, ch)
        }
        for i := 0; i < length_; i++ {
                fmt.Println(<-ch)
        }
        fmt.Printf("%.3fs elapsed\n", time.Since(start).Seconds())
}

func fetch(url string, ch chan<- string) {
        start := time.Now()
        resp, err := http.Get(url)
        if err != nil {
                ch <- fmt.Sprint(err)
                return
        }

        nbytes, err := io.Copy(ioutil.Discard, resp.Body)
        resp.Body.Close()
        if err != nil {
                ch <- fmt.Sprintf("While reading %s: %v", url, err)
                return
        }

        secs := time.Since(start).Seconds()
        ch <- fmt.Sprintf("%.3fs %7d %s", secs, nbytes, url)
}
```


<a id="org970e165"></a>

## 一个 web 服务器


<a id="orgb819268"></a>

### server1.go 返回请求路径

```go
// server1
package main

import (
        "fmt"
        "log"
        "net/http"
)

func main(){
        http.HandleFunc("/", handler)
        log.Fatal(http.ListenAndServe("localhost:8000",nil))
}

func handler (w http.ResponseWriter, r *http.Request){
        fmt.Fprintf(w, "URL.Path = %q\n", r.URL.Path)
}
```


<a id="orgb98c0f3"></a>

### server2.go 一个迷你回声和计数服务器

```go
//  server2 是一个迷你回声和计数服务器
package main

import (
        "fmt"
        "log"
        "net/http"
        "sync"
)

var mu sync.Mutex
var count int

func main(){
        http.HandleFunc("/", handler)
        http.HandleFunc("/count", counter)
        log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

func handler (w http.ResponseWriter, r *http.Request) {
        mu.Lock()
        count++
        mu.Unlock()
        fmt.Fprintf(w, "URL.Path = %q\n", r.URL.Path)
}

func counter(w http.ResponseWriter, r *http.Request) {
        mu.Lock()
        fmt.Fprintf(w, "Count %d\n", count)
        mu.Unlock()
}
```


<a id="orgc5fee37"></a>

### server3.go 处理程序回显HTTP 请求

```go
// 处理程序回显 HTTP 请求
package main

import (
        "fmt"
        "net/http"
        "log"
)

func main () {
        http.HandleFunc("/",handler)
        log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

func handler (w http.ResponseWriter, r *http.Request){
        fmt.Fprintf(w, "%s %s %s\n", r.Method, r.URL, r.Proto)
        for k,v := range r.Header {
                fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
        }
        fmt.Fprintf(w, "Host = %q\n", r.Host)
        fmt.Fprintf(w, "RemoteAddr = %q\n", r.RemoteAddr)
        if err := r.ParseForm(); err != nil {
                log.Print(err)
        }
        for k, v := range r.Form {
                fmt.Fprintf(w, "Form[%q] = %q\n", k, v)
        }
}
```


<a id="org10d1fde"></a>

### server4.go 同过网页显示 lissajous 图形

```go
//server4 通过浏览器显示 lissajous 图形
package main

import (
        "image"
        "image/color"
        "image/gif"
        "io"
        "log"
        "time"
        "math"
        "math/rand"
        "net/http"
)

var palette = []color.Color{color.White, color.Black}

const (
        whiteIndex = 0
        blackIndex = 1
)

func main() {
        rand.Seed(time.Now().UTC().UnixNano())
        http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
                lissajous(w)
        })
        log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

func lissajous(out io.Writer) {
        const (
                cycles  = 5
                res     = 0.001
                size    = 100
                nframes = 64
                delay   = 8
        )
        freq := rand.Float64() * 3.0
        anim := gif.GIF{LoopCount: nframes}
        phase := 0.0
        for i := 0; i < nframes; i++ {
                rect := image.Rect(0, 0, 2*size+1, 2*size+1)
                img := image.NewPaletted(rect, palette)
                for t := 0.0; t < cycles*2*math.Pi; t += res {
                        x := math.Sin(t)
                        y := math.Sin(t*freq + phase)
                        img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5), blackIndex)
                }
                phase += 0.1
                anim.Delay = append(anim.Delay, delay)
                anim.Image = append(anim.Image, img)
        }
        gif.EncodeAll(out, &anim)
}
```


<a id="org845f71a"></a>

## 控制流

```go
switch coinflip() {
case "heads":
        heads++
case "tails":
        tails++
default:
        fmt.Println("landed on edge!")
}
```

-   case 语句从上到下推演
-   默认 case 语句可以放在任何地方。
-   如果没有其他的case 语句符合条件，那么可选的默认case 语句将被执行
-   switch 语句不需要操作数,它就像一个case 语句列表，每条case 语句都是一个布尔表达式
-   这种形式称为无标签tagless 选择，它等价于 switch true
    
    ```go
    func Signum(x int) int {
            switch {
            case x > 0:
                    return +1
            default:
                    return 0
            case x < 0:
                    return 0
            }
    }
    ```


<a id="org8d42312"></a>

## 实现 Stringer interface

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// 
import (
        "fmt"
)

type human struct{
        name string
}

func (h *human) String () string {
        return fmt.Sprint(h.name, h.name)
}

func main () {
        var h = human{name:"liu"}
        fmt.Println("%s" ,&h)
}

```


<a id="org2bdb268"></a>

## bytecounter

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// ch7/bytecounter
import (
        "fmt"
)
type ByteCounter int

// 
func (c *ByteCounter) Write (p [] byte) (int, error) {
        *c += ByteCounter(len(p))
        return len(p), nil
}

func main () {
        var c ByteCounter
        c.Write([]byte("hello"))
        fmt.Println(c)

        c = 0
        name := "Dolly"
        fmt.Fprintf(&c, "hello, %s", name)
        fmt.Println(c)
}
```


<a id="org85e361b"></a>

## geometry

```go
// Filename: go.org[*Org Src go.org[ go ]*]
// coding:utf-8
// geometry
import (
        "fmt"
        "math"
)

type Point struct { X, Y float64}
// Path 是连接多个点的直线段
type Path []Point

// Distance 方法返回路径的长度
func (path Path) Distance () float64 {
        sum := 0.0
        for i := range path {
                if i > 0 {
                        sum += path[i - 1].Distance(path[i])
                }
        }
        return sum 
}

// 普通的函数
func Distance (p, q Point) float64 {
        return math.Hypot(p.X - q.X, p.Y - q.Y)
}

// Point 类型的方法
func (p *Point) Distance (q Point) float64 {
        return math.Hypot(p.X - q.X, p.Y - q.Y)
}

// 对点的x,y 成比例放大或缩小
func (p *Point) ScaleBy (factor float64) {
        p.X *= factor
        p.Y *= factor
}

// 格式化输出
func (p *Point) String () string{
        return fmt.Sprintf("X: %.2f, Y: %.2f", p.X, p.Y)
}

func main () {
        p := Point{1,2}
        q := Point{4,6}
        fmt.Println(Distance(p,q))
        fmt.Println(p.Distance(q))


        perim := Path{
                {1,1},
                {5,1},
                {5,4},
                {1,1},
        }
        fmt.Println(perim.Distance())

        // 指针接受者的方法
        r := &Point{1, 1}
        r.ScaleBy(10)
        fmt.Println(*r)
        fmt.Println(r)

        p1 := Point{1,2}
        pptr := &p1
        pptr.ScaleBy(10)
        fmt.Println(p1)
        fmt.Println(pptr)

        p2 := Point{1,3}
        (&p2).ScaleBy(10)
        fmt.Println(p2)
        fmt.Println(&p2)

        p3 := Point{1,4}
        p3.ScaleBy(10)
        fmt.Println(p3)
        fmt.Println(&p3)
}

```


<a id="org60aa654"></a>

## fib 函数

```go
// Filename: go.org[*Org Src go.org[ go ]*<2>]
// coding:utf-8
// fib

import (
        "fmt"
)

// fib
func fib (n int) (x int) {
        x, y := 0, 1
        for i := 1; i < n ; i++ {
                x, y = y, x + y 
        }
        return 
}

func main () {
        fmt.Println(fib(1000000))	
}

```


<a id="orgc9d03c2"></a>

## 数据类型


<a id="org01d8c9f"></a>

### basic type

-   基础类型


<a id="org7e9769b"></a>

### aggregate type

-   聚合类型


<a id="org88965c4"></a>

### reference type

-   引用类型


<a id="orga12f9c4"></a>

### interface type

-   接口类型
