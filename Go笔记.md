# 1 Go基础

## 1.1 语言基础

Go 内置了以下基本类型：

- 布尔
  - *bool*
- 字符串
  - *string*
- 整数
  - *int* *int8* *int16* *int32* *int64*
  - *uint* *uint8* *uint16* *uint32* *uint64*
- 字节
  - *byte* (*uint8* 的别名)
- Unicode
  - *rune* (*int32* 的别名，对标C语言中的char类型)
- 浮点
  - *float32* *float64*
- 复数
  - *complex64* *complex128*

Go语言中不能隐式的转化，必须显示地转化。  float64(num)

Go语言中首字母大小写是有区别的

```go
type Person struct {
    FirstName string  // 可被外部包访问
    lastName  string  // 仅包内可访问
}
```

**常量：const**

```go
const version = 1.0
```

**枚举：iota类型**

**go语言中 switch case 语句中不用break**

**函数：**  

```go
// func 函数名（参数列表） 返回值列表 
func ShowName() string{

}
func ShowInfo (bookName , author string , price float64) (bookInfo string , finalPrice float 64){
    
}
// 函数作为参数
func PrintBookInfo(do func(string , string ,float64)(string , float64),
                  bookName , author string ,price float64)

// 调用PrintBookInfo方法
PrintBookInfo( ShowInfo , "书名", "CYC" , 2983.3)
```

**err、panic、recover区别**

error:

- 在Go中，错误（errors）是通过返回值来表示的。
- Go 语言中 函数通常返回一个额外的 `error` 类型值，用于指示函数是否执行成功。
- 开发者通常通过检查返回的错误值来处理错误，并根据情况采取相应的行动。
- 错误的处理是通过 `if err != nil` 来实现的，如果 `err` 不为 `nil`，则表示有错误发生。

panic: 

- `panic` 是一种在运行时触发的紧急错误，它会导致程序中断。
- 当程序遇到无法继续执行的严重错误时，可以使用 `panic` 来中止程序的正常流程。
- `panic` 通常用于表示程序中发生了一些不可恢复的错误，例如数组越界、空指针解引用等。
- 当 `panic` 被调用时，程序会立即停止执行当前函数的代码，并开始向调用栈的上层函数传播，直到达到程序的顶层。

recover

​		在Go语言中，`recover` 是用于捕获 panic 异常的内建函数。当程序发生 panic 时，`recover` 可以用于防止程序崩溃，并且允许程序进行一些清理工作或记录错误信息。`recover` 只有在延迟函数（deferred function）内调用才会生效。





## 1.2 指针

Go语言中的指针不可以运算，而C语言中的指针是可以运算的。

```go
var price int  = 68
var ptr *int  = &price
```

值传递和引用传递

```go
func byValue(price int){
	price += 20
}
func byRef(price *int){
	*price += 20
}

byValue(price)
byRef(&price)
```



## 1.3 数组、切片（slice）、Map

初始化数组

```go
var array1 [6]string
array2 := [3]string{"火锅","烧烤","家常菜"}
array3 := [...]string{"火锅","烧烤","家常菜"}
```

在Go语言中数组是值类型！其它语言中大部分是引用类型

如果想要引用类型，就得使用到指针！

```go
// 值类型！ 
func PrintFood1(foods [3]string){
	foods[2] = "大饼子"
	fmt.Println(foods)
}
// 指针
func PrintFood2(foods *[3]string){
	foods[2] = "大饼子"
	fmt.Println(foods)
}
PrintFood1(foods)
PrintFood2(&foods)
```



**切片**（引用类型）

切片是对数组的一个引用，它包含了一个指向数组的指针、切片的长度和容量信息。创建切片：

```go
 	// 创建一个切片
    var mySlice []int
    fmt.Println("Empty Slice:", mySlice)

    // 使用make函数创建切片，指定长度和容量
    mySlice2 := make([]int, 5, 10)
    fmt.Println("Slice with make:", mySlice2)

    // 直接初始化切片
    mySlice3 := []int{1, 2, 3, 4, 5}
    fmt.Println("Initialized Slice:", mySlice3)
```



```go
// 先定义一个数组array3
array3 := [...]string{"火锅","烧烤","家常菜","家常菜1","家常菜2"}
// 使用切片Slice
slice := array3[1:3]  // 左闭右开
slice1 := array2[1:]  // 1后面的都要
slice2 := array2[:3]  // 3前面的
slice3 := array2[:]  // 全要
// 使用make创建切片
s1 := make([]string , 8)
s2 := make([]string , 8 ,16)
```

![image-20231122121411481](Go笔记.assets/image-20231122121411481.png)

**slice中添加数据元素**

```go
slice8 = append(slice8 , "蛋炒饭")
```

**Go语言中数组和切片的区别有哪些？**

1. **长度固定 vs. 长度可变：**
   - **数组：** 数组的长度是固定的，一旦创建就不能更改。
   - **切片：** 切片的长度是可变的，它可以根据需要动态增长或缩小。
2. **内存管理：**
   - **数组：** 数组是值类型，将一个数组赋值给另一个数组会复制整个数组的数据。
   - **切片：** 切片是引用类型，它包含一个指向底层数组的指针、长度和容量信息。多个切片可以引用相同的底层数组，因此修改一个切片会影响其他引用同一底层数组的切片。
3. **初始化：**
   - **数组：** 可以通过指定固定大小并用元素值初始化数组。
   - **切片：** 通常使用 `make` 函数或切片字面量初始化。切片的长度可以在初始化时指定，也可以后续动态增加。
4. **传递参数：**
   - **数组：** 作为参数传递给函数时，会进行值拷贝。
   - **切片：** 作为参数传递给函数时，实际上传递的是切片的引用，因此在函数内对切片的修改会影响到原始切片。
5. **迭代：**
   - **数组：** 使用索引进行迭代。
   - **切片：** 使用 `range` 关键字进行迭代。



**Map**

map 是哈希map，它是无序的，不保证顺序

创建map的方式

```go
1. m := make(map[string]string)
2. var m2 map[string]string
// 对于已声明但未初始化的map，不能直接添加键值对，否则会引发运行时错误。因此，在使用之前，最好使用make函数进行初始化。
m2 = make(map[string]string)
3. m3 := map[string]string{
    "鲁菜"：“九转大肠”,
	"川菜"：“麻婆豆腐”,
}
4. m4:=map[string]map[int]string{
    
}
```

## 1.4 结构体、方法

面向接口编程！！

```go
type Food struct{
	No int32
	Name string
}
```

方法：属于一个结构体的，类似于C++中的this，PY中的self

```go
func (f *Food) Show(){
	fmt.Pringln(f.Name)
}
```

GO语言中函数和方法的区别？


在Go语言中，函数（Function）和方法（Method）有一些关键的区别：

1. **定义位置：**
   - **函数：** 函数是独立的代码块，可以在任何地方定义，不依赖于特定的类型。
   - **方法：** 方法是与特定类型关联的函数。它们必须在类型的定义所在的包中声明。
2. **关联类型：**
   - **函数：** 函数不与任何类型直接关联。
   - **方法：** 方法与一个类型关联。它是在类型的定义中声明的函数。
3. **调用方式：**
   - **函数：** 函数通过包名直接调用，例如`packageName.FunctionName(arguments)`。
   - **方法：** 方法是与类型的实例关联的，通过实例（或者类型）调用，例如`instance.MethodName(arguments)`。
4. **语法：**
   - **函数：** 函数的声明没有接收者（receiver），它们是独立于任何类型的。
   - **方法：** 方法的声明包含一个接收者，即方法的调用者。

## 1.5 接口

​		在Go语言中，接口（interface）是一种定义对象行为的方式，而不关心对象的具体类型。一个对象只要实现了接口中定义的方法，就被视为实现了该接口。接口提供了一种抽象的方式，使得代码更加灵活和可复用。

```go
// 定义一个接口
type Shape interface {
    Area() float64
}

// 定义一个结构体（圆）
type Circle struct {
    Radius float64
}

// Circle 结构体实现 Shape 接口的 Area 方法
func (c Circle) Area() float64 {
    return 3.14 * c.Radius * c.Radius
}

// 定义一个结构体（矩形）
type Rectangle struct {
    Width  float64
    Height float64
}

// Rectangle 结构体实现 Shape 接口的 Area 方法
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

```



## 1.6 函数式编程



```go
// 匿名函数赋值给一个变量
show2 := func(){
    ...
}
```





## 1.7 延迟调用 defer

​		Go语言中进行资源释放等功能，通常使用defer函数来实现。

​		在Go语言中，`defer` 语句用于确保一个函数调用在程序执行结束时执行。通常，我们使用 `defer` 来释放资源（例如关闭文件或网络连接）或执行一些清理操作，以确保在函数执行结束时执行这些任务。`defer` 语句的执行顺序是“后进先出”（Last In, First Out，LIFO）的，也就是说，最后一个 `defer` 语句将最先执行，依此类推。



 ## 1.8 通道与协程



channel（FIFO）：

​		CSP模型：通信双方抽象出中间层，数据的流转由中间层控制，通信双方只负责数据的收发，这样就实现了数据的共享。（通过通信来共享内存）

```go
ch := make(chan string)  // 无缓冲通道。必须等有人消费了才能继续运行
ch2 := make(chan string , 6) // 有缓冲通道。6：可缓冲数据的容量
ch <- "回锅肉" // 往chan里写数据
<- ch // 读数据
close(ch) // 关闭通道

```

**WaitGroup**



**Select**

​		类似与switch，case， default，如果有default的情况下，直接走default中的代码

​		select语句会阻塞在case里，知道监听到一个可以执行的IO操作



**锁（Lock）**、sync.Mutex、sync.RWMutex

```go
type Goods struct{
	v map[string] int
	m sync.Mutex
}
func (g *Goods) Inc(key string , num int){
    g.m.Lock()
    defer g.m.Unlock()
    fmt...
    g.v[key]++
    fmt...
}
```



## 1.9 包管理

go mod tidy ：清理未使用的包信息

go get -u 依赖包 ：升级包

go mod init 名称：新建mod的名字

go get 依赖@版本

go build ./...



# 2 Go Web 开发

## 2.1 Gin框架

官网：https://gin-gonic.com/zh-cn/docs/

```go
r := gin.Default()
r.GET("/" , func(c *gin.Context){
    c.JSON(http.StatusOK,gin.H{
        "msg":"aaaaa"        
    })
})
r.Run()

// gin.H 本质上是一个map  key：string v：interface{}
```

gin.Default()

gin.New()

它们分别做了什么？有什么区别？

![image-20231127201500187](Go笔记.assets/image-20231127201500187.png)

![image-20231127201630111](Go笔记.assets/image-20231127201630111.png)

































​    













