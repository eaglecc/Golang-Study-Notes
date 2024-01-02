# 1 Go基础语法

## 1.1 数据类型

**1.使用值为 nil 的 slice、map会发生啥**

允许对值为 nil 的 slice 添加元素，但对值为 nil 的 map 添加元素，则会造成运行时 panic。

```go
// map 错误示例
func main() {
    var m map[string]int
    m["one"] = 1  // error: panic: assignment to entry in nil map
    // m := make(map[string]int)// map 的正确声明，分配了实际的内存
}    

// slice 正确示例
func main() {
 var s []int
 s = append(s, 1)
}
```

 **2.访问 map 中的 key，需要注意啥**

当访问 map 中不存在的 key 时，Go 则**会返回元素对应数据类型的零值，比如 nil、’’ 、false 和 0，**取值操作总有值返回，故不能通过取出来的值，来判断 key 是不是在 map 中。

检查 key 是否存在可以用 map 直接访问，检查返回的第二个参数即可。

```go
// 错误的 key 检测方式
func main() {
 x := map[string]string{"one": "2", "two": "", "three": "3"}
 if v := x["two"]; v == "" {
  fmt.Println("key two is no entry") // 键 two 存不存在都会返回的空字符串
 }
}

// 正确示例
func main() {
 x := map[string]string{"one": "2", "two": "", "three": "3"}
 if _, ok := x["two"]; !ok {
  fmt.Println("key two is no entry")
 }
}
```

**3.string 类型的值可以修改吗**

**不能，**尝试使用索引遍历字符串，来更新字符串中的个别字符，是不允许的。

string 类型的值是只读的二进制 byte slice，如果真要修改字符串中的字符，**将 string 转为 []byte 修改后，再转为 string 即可。**

```go
// 修改字符串的错误示例
func main() {
 x := "text"
 x[0] = "T"  // error: cannot assign to x[0]
 fmt.Println(x)
}


// 修改示例
func main() {
 x := "text"
 xBytes := []byte(x)
 xBytes[0] = 'T' // 注意此时的 T 是 rune 类型
 x = string(xBytes)
 fmt.Println(x) // Text
}
```

**4.switch 中如何强制执行下一个 case 代码块**

switch 语句中的 case 代码块会默认带上 break，但可以使用 fallthrough 来强制执行下一个 case 代码块。

```go
func main() {
     isSpace := func(char byte) bool {
          switch char {
          case ' ': // 空格符会直接 break，返回 false // 和其他语言不一样
          // fallthrough // 返回 true
          case '\t':
           return true
          }
          return false
     }
     fmt.Println(isSpace('\t')) // true
     fmt.Println(isSpace(' ')) // false
}
```

**5.如何从 panic 中恢复**

在一个 **defer 延迟执行的函数中调用 recover ，它便能捕捉/中断 panic。**

```go
// 错误的 recover 调用示例
func main() {
 recover() // 什么都不会捕捉
 panic("not good") // 发生 panic，主程序退出
 recover() // 不会被执行
 println("ok")
}

// 正确的 recover 调用示例
func main() {
 defer func() {
  fmt.Println("recovered: ", recover())
 }()
 panic("not good")
}
```

**6.简短声明的变量需要注意啥**

- 简短声明的变量只能在**函数内部**使用
- **struct 的变量字段不能使用 := 来赋值**
- 不能用简短声明方式来**单独为一个变量重复声明**， := 左侧至少有一个新变量，才允许多变量的重复声明

**7.range 迭代 map是有序的吗**

**无序的。**Go 的运行时是有意打乱迭代顺序的，所以你得到的迭代结果可能不一致。但也并不总会打乱，得到连续相同的 5 个迭代结果也是可能的。

## 1.2 defer

**1.recover的执行时机**

**recover 必须在 defer 函数中运行**。recover **捕获的是祖父级调用时的异常**，**直接调用时无效。**

```go
func main() {
    recover() // 无效
    panic(1)
}
```

直接 defer 调用也是无效。

```go
func main() {
    defer recover()
    panic(1)
}
```

defer 调用时多层嵌套依然无效。

```go
func main() {
    defer func() {
        func() { recover() }()
    }()
    panic(1)
}
```

**必须在 defer 函数中直接调用**才有效。

```
func main() {
    defer func() {
        recover()
    }()
    panic(1)
}
```

**2.闭包**

闭包（Closure）是一种函数值，它引用了在其外部作用域中定义的变量。**闭包允许这些变量在闭包函数内被访问和修改，**即使这些变量在闭包函数被调用时不再处于其原始作用域。

```go
func closureFunc() func() int {
    // 在闭包函数内定义变量
    x := 0
    // 返回一个匿名函数
    return func() int {
        // 访问并修改外部作用域的变量
        x++
        return x
    }
}
func main() {
    // 创建一个闭包函数
    myClosure := closureFunc()
    // 调用闭包函数，它会记住在创建时的上下文
    fmt.Println(myClosure())  // 输出：1
    fmt.Println(myClosure())  // 输出：2
}
```

在上面的例子中，`closureFunc` 返回一个匿名函数，这个匿名函数可以访问并修改 `closureFunc` 内的变量 `x`。每次调用 `myClosure` 时，它都会保留对相同的 `x` 变量的引用，并且 `x` 的值在每次调用时都会递增。

闭包在Go中常用于实现函数式编程的一些特性，例如函数柯里化、延迟执行等。通过闭包，可以将函数作为一等公民传递，更灵活地处理程序逻辑。

**3.闭包错误引用同一个变量问题怎么处理**

在每轮迭代中生成一个局部变量 i 。如果没有 i := i 这行，将会打印同一个变量5，这是因为`defer`语句内部引用了循环变量`i`，当`defer`语句执行时，`i`已经变成了5。因此，无论循环执行了多少次，`defer`语句最终输出的都是5。

```go
func main() {
    for i := 0; i < 5; i++ {
        i := i
        defer func() {
            println(i)
        }()
    }
}
```

或者是通过函数参数传入 i 。

```go
func main() {
    for i := 0; i < 5; i++ {
        defer func(i int) {
            println(i)
        }(i)
    }
}
```

**4.在循环内部执行defer语句会发生啥**

defer 在函数退出时才能执行，在 for 执行 defer 会导致资源延迟释放。

```go
func main() {
    for i := 0; i < 5; i++ {
        func() {
            f, err := os.Open("/path/to/file")
            if err != nil {
                log.Fatal(err)
            }
            defer f.Close() // 匿名函数func()退出时执行defer语句
        }()
    }
}
```

func 是一个局部函数，在局部函数里面执行 defer 将不会有问题。由于 `defer` 语句是后进先出的，所以在循环结束时，会按照后进先出的顺序关闭文件。最后一次迭代会最先执行，然后是倒数第二次，以此类推。







## 1.3 http

**1.你是如何关闭 HTTP 的响应体的**

直接在处理 HTTP 响应错误的代码块中，直接关闭非 nil 的响应体；手动调用 defer 来关闭响应体。

```go
// 正确示例
func main() {
 resp, err := http.Get("http://www.baidu.com")
 // 关闭 resp.Body 的正确姿势
 if resp != nil {
    defer resp.Body.Close()
 }

 checkError(err)
 defer resp.Body.Close()

 body, err := ioutil.ReadAll(resp.Body)
 checkError(err)

 fmt.Println(string(body))
}
```

**2.你是否主动关闭过http连接，为啥要这样做**

有关闭，不关闭会程序可能会消耗完 socket 描述符。有如下2种关闭方式：

- 直接**设置请求变量的 Close 字段值为 true**，每次请求结束后就会主动关闭连接。或者**设置 Header 请求头部选项 Connection: close，然后服务器返回的响应头部也会有这个选项，此时 HTTP 标准库会主动断开连接**

```go
// 主动关闭连接
func main() {
 req, err := http.NewRequest("GET", "http://golang.org", nil)
 checkError(err)

 req.Close = true
 //req.Header.Add("Connection", "close") // 等效的关闭方式

 resp, err := http.DefaultClient.Do(req)
 if resp != nil {
  defer resp.Body.Close()
 }
 checkError(err)

 body, err := ioutil.ReadAll(resp.Body)
 checkError(err)

 fmt.Println(string(body))
}
```

你可以创建一个自定义配置的 HTTP transport 客户端，用来取消 HTTP 全局的复用连接。

```go
func main() {
 tr := http.Transport{DisableKeepAlives: true}
 client := http.Client{Transport: &tr}

 resp, err := client.Get("https://golang.google.cn/")
 if resp != nil {
  defer resp.Body.Close()
 }
 checkError(err)

 fmt.Println(resp.StatusCode) // 200

 body, err := ioutil.ReadAll(resp.Body)
 checkError(err)

 fmt.Println(len(string(body)))
}
```

**3.解析 JSON 数据时，默认将数值当做哪种类型**

在 encode/decode JSON 数据时，Go 默认会将数值当做 float64 处理。

```go
func main() {
     var data = []byte(`{"status": 200}`)
     var result map[string]interface{}

     if err := json.Unmarshal(data, &result); err != nil {
     log.Fatalln(err)
}
```

解析出来的 200 是 float 类型。



## 1.4 Goroutine

**1.说出一个避免Goroutine泄露的措施**

可以通过 context 包来避免内存泄漏。

```go
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    ch := func(ctx context.Context) <-chan int {
        ch := make(chan int)
        go func() {
            for i := 0; ; i++ {
                select {
                case <- ctx.Done():
                    return
                case ch <- i:
                }
            }
        } ()
        return ch
    }(ctx)

    for v := range ch {
        fmt.Println(v)
        if v == 5 {
            cancel()
            break
        }
    }
}
```

下面的 for 循环停止取数据时，就用 cancel 函数，让另一个协程停止写数据。如果下面 for 已停止读取数据，上面 for 循环还在写入，就会造成内存泄漏。





















































































