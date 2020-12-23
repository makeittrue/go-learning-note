# Task09:异常处理

## 基础知识

Go中有错误和异常两个不同的概念，非常容易混淆。很多的程序猿习惯性的讲一切非正常情况都看作错误，而不区分错误和异常，即使程序中可能有异常抛出，也将异常即使捕获并转换成错误。从表面上看，把一切都视为错误的思路更简单，而异常的引入仅仅增加了额外的复杂度。

但实际上并非如此，总所周知，Golang遵循“少即是多”的设计哲学，追求简洁优雅，就是说如果异常价值不大，就不会将异常加入到语言特性中。

Golang中引入error接口类型作为错误处理的标准模式，如果函数要返回错误，则返回值类型列表中肯定包含error。error处理过程类似于C语言中的错误码，可逐层返回，直到被处理。

Golang中引入两个内置函数panic和recover来触发和终止异常处理流程，同时引入关键字defer来延迟执行defer后面的函数。

一直等到包含defer语句的函数执行完毕时，延迟函数（defer后的函数）才会被执行，而不管包含defer语句的函数是通过return的正常结束，还是由于panic导致的异常结束。你可以在一个函数中执行多条defer语句，它们的执行顺序与声明顺序相反。

当程序运行时，如果遇到引用空指针、下标越界或显式调用panic函数等情况，则先触发panic函数的执行，然后调用延迟函数。调用者继续传递panic，因此该过程一直在调用栈中重复发生：函数停止执行，调用延迟执行函数等。如果一路在延迟函数中没有recover函数的调用，则会到达该携程的起点，该携程结束，然后终止其他所有携程，包括主携程（类似于C语言中的主线程，该携程ID为1）。

错误和异常从Golang机制上讲，就是error和panic的区别。很多其他语言也一样，比如C++/Java，没有error但有errno，没有panic但有throw。

Golang错误和异常是可以互相转换的：

1. 错误转异常，比如程序逻辑上尝试请求某个URL，最多尝试三次，尝试三次的过程中请求失败是错误，尝试完第三次还不成功的话，失败就被提升为异常了。
2. 异常转错误，比如panic触发的异常被recover恢复后，将返回值中error类型的变量进行赋值，以便上层函数继续走错误处理流程。

### 错误和异常的区别

错误指的是可能出现问题的地方出现了问题，比如打开一个文件时失败，这种情况在人们的意料之中；而异常指的是不应该出现问题的地方出现了问题，比如引用了空指针，这种情况在人们的意料之外，由此可见，**错误是业务过程的一部分，而异常不是。**

## 1. error

Go语言内置了一个简单的错误接口作为一种错误处理机制，接口定义如下：

```go
type error interface {
	Error() string
}
```

它包含一个 `Error()` 方法，返回值为`string`

Go的error构造有两种方式，分别是

**第一种**：errors.New()

```go
err := errors.New("This is an error")
if err != nil {
  fmt.Print(err)
}
```

**第二种**：fmt.Errorf()

```go
err := fmt.Errorf("This is an error")
if err != nil {
  fmt.Print(err)
}
```

除了直接使用Go自带的方法，还可以自定义错误。下面以自然数函数作为例子：

```go
type NotNature float64

func (err NotNature) Error() string {
 return fmt.Sprintf("自然数为大于或等于0的数: %v", float64(err))
}

func Nature(x float64) (float64,error) {
  if x<0 {
   return 0,NotNature(x)
  } else {
  return x,nil
  }
}

func main() {
  fmt.Println(Nature(1))
  fmt.Println(Nature(-1))
}
```

需要注意一下几点：

1.如果函数需要处理异常，通常将error作为多值返回的最后一个值，返回的error值为nil则表示无异常，非nil则是有异常。

2.一般先用if语句处理error!=nil，正常逻辑放if后面。

Go语言的error代表的并不是真“异常”，只是通过返回error来表示错误信息，换句话说，不是运行时错误范围预定义的错误，某种不符合期望的行为并不会导致程序无法运行（自然数函数例子），都应使用error进行异常处理。当程序出现重大错误，如数组越界，才会将其当成真正的异常，并用**panic**来处理。

## 2. panic

Go不使用try...catch方法来处理异常，而是使用panic和recover

先上代码举一个简单的例子

```go
func main() {
  fmt.Println("Hello,Go!")
  panic(errors.New(" i am a error"))
  fmt.Println("hello,again!")
}
```

输出：

```go
Hello,Go!
panic:  i am a error

goroutine 1 [running]:
main.main()
	~/error.go:12 +0xb5
exit status 2
```

可以看到，panic后面的程序不会被执行了。但是我们捕捉异常并不是为了停止程序(一般情况)，而是为了让程序能正常运行下去，这时候就到recover出场了。

```go
package main

import "fmt"

func main(){
  defer func(){
    fmt.Println("我是defer里面第一个打印函数")
    if err:=recover();err!=nil{
        fmt.Println(err)
    }
    fmt.Println("我是defer里面第二个打印函数")
  }()
  f()
}

func f(){
  fmt.Println("1")
  panic("我是panic")
  fmt.Println("2")
}
```

输出：

```shell
1
我是defer里面第一个打印函数
我是panic
我是defer里面第二个打印函数
```

可以看到，f函数一开始正常打印，当遇到panic，就跳到defer函数，执行defer函数里的内容

需要注意的是，defer函数里打印的err其实就是panic里面的内容。

下面详细介绍一下panic和recover的原理。首先来看一下panic和recover的官方定义

```go
func panic(v interface{})
```

> 内置函数panic会停止当前goroutine的正常执行。当函数F调用panic时，F的正常执行立即停止。任何被F延迟执行的函数都将以正常的方式运行，然后F返回其调用者。对调用方G来说，对F的调用就像调用panic一样，终止G的执行并运行任何延迟的函数。直到执行goroutine中的所有函数都按逆序停止。此时，程序将以非0退出代码终止。此终止序列称为panicking，可由内置函数recover控制。

```go
func recover() interface{}
```

> recover内置函数允许程序管理panicking的goroutine的行为。在defer函数（但不是它调用的任何函数）内执行恢复调用，通过恢复正常执行来停止panicking序列，并检索传递给panic调用的错误值。如果在defer函数之外调用recover，则不会停止panicking的序列。在这种情况下，或者当goroutine不panicking时，或者提供给panic的参数是nil，recover返回nil。因此，recover的返回值报告goroutine是否panicking

**ps**：defer和recover必须在panic之前定义，否则无效。

## 3. 源码分析

errors.New的定义如下

```go
// src/errors/errors.go

// New returns an error that formats as the given text.
// Each call to New returns a distinct error value even if the text is identical.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```

1.New函数返回格式为给定文本的错误

2.即使文本是相同的，每次对New的调用都会返回一个不同的错误值。

## 参考

1. [Golang错误和异常处理的正确姿势](read://https_www.cnblogs.com/?url=https%3A%2F%2Fwww.cnblogs.com%2Fzhangboyu%2Fp%2F7911190.html)