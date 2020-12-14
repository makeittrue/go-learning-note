# Task01: Go初探

## 1. go语言介绍

### 1.1 go的由来

> Go是从2007年末由Robert Griesemer, Rob Pike, Ken Thompson主持开发，后来还加入了Ian Lance Taylor, Russ Cox等人，并最终于2009年11月开源，在2012年早些时候发布了Go 1稳定版本。现在Go的开发已经是完全开放的，并且拥有一个活跃的社区。

### 1.2 go的特性

> - 自动垃圾回收
> - 更丰富的内置类型
> - 函数多返回值
> - 错误处理
> - 匿名函数和闭包
> - 类型和接口
> - 并发编程
> - 反射
> - 语言交互性

### 1.3 go代码案例解读

> - 包声明
> - 引入包
> - 函数
> - 变量
> - 语句 & 表达式
> - 注释

```go
package main 

import "fmt"  
func main() { 
   /* Always Hello, World! */
   fmt.Println("Hello, World!")
}
```

- **解析:**

1.  package main定义了包名。**必须**在源文件中非注释的第一行指明这个文件属于哪个包。package main表示一个可独立执行的程序，每个 Go 应用程序都包含一个名为 main 的包。

2. import "fmt"告诉编译器程序运行需要用fmt包。

3. func main() 是程序开始执行的函数，main 函数是每一个可执行程序所必须包含的，一般来说都是在启动后第一个执行的函数**（如果有 init() 函数则会先执行该函数）**。

4. **=={}中"{"不可以单独放一行。==**

5. 注释： 

   - // 单行注释      

   - /* 块注释（ 多行注释） 不可以嵌套使用，一般用于包的文档描述或注释成块的代码片段。*/   

6. fmt.Println(...) 将字符串输出到控制台，并在最后自动增加换行字符 \n。等价于 fmt.Print("hello, world\n") 。



## 2. go环境配置

### 2.1 IDE选择

> IDE需知：vscode与goland，推荐使用vscode，vscode免费，goland收费。
>
> vscode地址：https://code.visualstudio.com/
>
> goland地址：[https://www.jetbrains.com/go/](https://www.jetbrains.com/go/)

### 2.2 go安装包

> 下载地址：
>
> 1. [https://golang.org/](https://golang.org/)    
> 2. [Go下载 - Go语言中文网 - Golang中文社区 (studygolang.com)](https://studygolang.com/dl)

### 2.3 go配置

> - 不管是哪个系统，需要配置go安装环境到path环境变量里面。   **go/bin目录添加到path中**
> - goland只需要选择go对应的sdk配置即可，也就是把go安装的路径添加进去，使用安装的go环境进行编译，不需要安装任何额外插件。
> - vscode需要安装额外插件，在插件市场里面直接搜索go，随后进行安装即可。在安装的时候容易出现下载失败问题，此时需要更换为国内源，如下设置：

```go
windows设置：
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

```go
MacOS or Linux
export GO111MODULE=on
export GOPROXY=https://goproxy.cn
```

![image-20201214210543171](C:\Users\Han Bai\AppData\Roaming\Typora\typora-user-images\image-20201214210543171.png)

---

