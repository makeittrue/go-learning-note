# Task03：变量、常量、枚举

# 1. 变量

> 变量，计算机语言能存储计算结果或表示值的抽象概念。可以通过变量名访问，变量名由字母、数字、下划线组成，其中首个字符不能为数字。

- 变量声明一般形式

```go
//格式
var identifier type  // var 标识符 类型
var identifier1, identifier2 type  //var 标识符1, 标识符2 type
```

```go
var identifier int = 100
var identifier1,identifier2 int = 200,300
fmt.Println(identifier+identifier1+identifier2) //600
```

## 1.1 变量声明的四种方式

1. **指定变量类型，如果没有初始化，则变量默认为零值。**

   > - 数值类型（包括complex64/128）为 **0**
   > - 布尔类型为 **false**
   > - 字符串为 **""**（空字符串）

```go
//格式
var name type
name = value
```

```go
// 没有初始化就为零值
var b int
fmt.Println(b) //0

// bool 零值为 false
var c bool
fmt.Println(c) //false

//string默认为""
var d string
fmt.Println(d)
```

```go
//以下几种类型为 nil：
var a *int   //<nil>
var a []int 
var a map[string] int
var a chan int
var a func(string) int
var a error // error 是接口
```

2. **可根据值自行判断类型**

```go
// 声明一个变量并初始化
var a = "RUNOOB"  //可根据值自行判断类型
fmt.Println(a)  //RUNOOB
```

3. **省略 var，使用 :=来进行高效快速声明变量**

> := 的左侧必须**有**新的变量

```go
//格式
newname := value
```

```go
var intVal int 
intVal :=1 // 这时候会产生编译错误,因为intVal不是新的变量
intVal,intVal1 := 1,2 // 此时不会产生编译错误，因为有声明新的变量，因为 := 是一个声明语句
```

```go
f := "Runoob" // var f string = "Runoob"   简写
fmt.Println(f) //Runoob
```

4. **多变量声明**

```go
//类型相同多个变量, 非全局变量   以下格式均独立
var vname1, vname2, vname3 type
vname1, vname2, vname3 = v1, v2, v3

var vname1, vname2, vname3 = v1, v2, v3 // 和 python 很像,不需要显示声明类型，自动推断


// 这种因式分解关键字的写法一般用于声明全局变量
var (
    vname1 v_type1
    vname2 v_type2
)
```

```go
package main

var x, y int
var (  // 这种因式分解关键字的写法一般用于声明全局变量
    a int
    b bool
)

//可以自动匹配默认类别
var c, d int = 1, 2
var e, f = 123, "hello"

//这种不带声明格式的只能在函数体中出现
//g, h := 123, "hello"

func main(){
    g, h := 123, "hello"
    println(x, y, a, b, c, d, e, f, g, h) //0 0 0 false 1 2 123 hello 123 hello
}
```

## 2.2 注意

1. "：=" 赋值操作符,高效创建新变量，初始化声明：a := 50 或 b := false，a 和 b 的类型（int 和 bool）将由编译器自动推断。
2. :=  是使用变量的首选形式，但是它只能被用在函数体内，而不可以用于全局变量的声明与赋值。
3. 在相同的代码块中，我们不可以再次对于相同名称的变量使用初始化声明，但可以赋值；
4. 声明了一个局部变量却没有在相同的代码块中使用它，同样会得到编译错误
5. 全局变量可以声明但不用。
6. _ 实际上是一个只写变量，你不能得到它的值。这样做是因为 Go 语言中必须使用所有被声明的变量，但有时你并不需要使用从一个函数得到的所有返回值。

```go
//空白标识符在函数返回值时的使用：
func main() {
  _,numb,strs := numbers() //只获取函数返回值的后两个
  fmt.Println(numb,strs)
}

//一个可以返回多个值的函数
func numbers()(int,int,string){
  a , b , c := 1 , 2 , "str"
  return a,b,c
}
```



# 2. 常量

> 常量是一个简单值的标识符，在程序运行时，不会被修改的量。数据类型只可以是**布尔型、数字型（整数型、浮点型和复数）和字符串型**。
>
> 可以省略类型说明符 [type]，因为编译器可以根据变量的值来推断其类型。

- **常量的定义格式**

```go
const identifier [type] = value

const b string = "abc" //显式类型定义：
const b = "abc" //隐式类型定义：
```

- **多个相同类型的声明**

```go
const c_name1, c_name2 = value1, value2
```

```go
//常用于枚举
const (
    //数字 0、1 和 2 分别代表未知性别、女性和男性。
    Unknown = 0
    Female = 1
    Male = 2
) 
```

```go
const(
	he = 100
	ha = 200
	xi  //此时xi与ha等值 
)
fmt.Println(he,ha,xi) //100 200 200
```

- **常量的特性**

> 可以用len(), cap(), unsafe.Sizeof()函数计算表达式的值。常量表达式中，函数必须是内置函数，否则编译不过：

```go
package main

import "unsafe"
const (
    a = "abc"
    b = len(a)
    c = unsafe.Sizeof(a)
)

func main(){
    println(a, b, c)  //abc 3 16
}
```

> **字符串类型的 unsafe.Sizeof() 为什么一直是16？**
>
> ​       实际上字符串类型对应一个结构体，该结构体有两个域，第一个域是指向该字符串的指针，第二个域是字符串的长度，每个域占8个字节，但是并不包含指针指向的字符串的内容，这也就是为什么sizeof始终返回的是16。

- **特殊常量 iota**

> **iota**，特殊常量，**行索引常量**，可认为是可以被编译器修改的常量。在 const关键字中第一次出现时将被重置为 0(const 内部的第一行之前)，**之后const 中每新增一行常量声明将使 iota 计数一次。**其常被用于枚举值。

```go
const (
    a = 5
    b = iota
    c 
)

fmt.Println(a,b,c)  //5 1 2
```

```go
const (
    a = iota   //0
    b          //1
    c          //2
    d = "ha"   //独立值，iota += 1
    e          //"ha"   iota += 1
    f = 100    //iota +=1
    g          //100  iota +=1
    h = iota   //7,恢复计数
    i          //8
)
fmt.Println(a,b,c,d,e,f,g,h,i)  // 0 1 2 ha ha 100 100 7 8
```

```go
const ( //  <<n==*(2^n)   左移几位，即为结果 乘 2的n次
    i=1<<iota  // i=1：左移 0 位,不变仍为 1;
    j=3<<iota  // j=3：左移 1 位,变为二进制 110, 即 6;
    k          // k=3：左移 2 位,变为二进制 1100, 即 12;
    l          // l=3：左移 3 位,变为二进制 11000,即 24。
)

func main() {
    fmt.Println("i=",i)  //1
    fmt.Println("j=",j)  //6
    fmt.Println("k=",k)  //12
    fmt.Println("l=",l)  //24
}
```



# 3. 枚举

> 枚举，将变量的值一一列举出来，变量只限于列举出来的值的范围内取值。Go语言中没有枚举这种数据类型的，但是可以使用const配合iota模式来实现

### 3.1.1 普通枚举

```go
const (
	a = 0
	b = 1
	c = 2
	d = 3
)

```

### 3.1.2 自增枚举

```go
const (
	a = iota  //0  默认开始值是0，
	b		  //1
	c         //2
)
const (
	e, f = iota, iota //e=0, f=0    默认开始值是0，const中每增加一行加1,同行值相同
	g    = iota       //g=1
)

const (
    a = iota    //0     //若中间中断iota，必须显式恢复。
    b           //1     //即使不恢复，iota值也会默认在改变
    c = 100     //100
    d           //100
    e = iota    //4
)
```

