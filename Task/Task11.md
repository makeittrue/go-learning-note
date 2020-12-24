# Task11：反射机制

## 一、什么是反射机制？

百度百科上的定义为：

> 反射的概念是由Smith在1982年首次提出的，主要是指程序可以访问、检测和修改它本身状态或行为的一种能力。
>
> 这一概念的提出很快引发了计算机科学领域关于应用反射性的研究。它首先被程序语言的设计领域所采用，并在Lisp和[面向对象](https://baike.baidu.com/item/面向对象/2262089)方面取得了成绩。其中LEAD/LEAD++ 、OpenC++ 、MetaXa和OpenJava等就是基于反射机制的语言。

维基百科上的定义为：

> 在计算机科学中，反射是指计算机程序在运行时（Run time）可以访问、检测和修改它本身状态或行为的一种能力。用比喻来说，反射就是程序在运行的时候能够“观察”并且修改自己的行为。

不同语言的反射模型不尽相同，有些语言还不支持反射。[《Go 语言圣经》](http://shouce.jb51.net/gopl-zh/ch1/ch1-01.html)中是这样定义反射的：

> Go 语言提供了一种机制在运行时更新变量和检查它们的值、调用它们的方法，但是在编译时并不知道这些变量的具体类型，这称为反射机制。

### 1. 为什么要用反射？

需要用反射的两个常见场景：

- 有时候需要编写一个函数，但是并不知道传给你的参数类型是什么，可能是没约定好，也可能是传入的类型很多，这些类型并不能统一表示，这是反射机制就派上了用场。

- 有时候需要根据某些条件决定调用哪个函数，比如根据用户的输入来决定，这时就需要对函数和函数的参数进行反射，在运行期间动态地执行函数。

  

在讲反射的原理以及如何运用之前，说明几点不使用反射的理由：

- 与反射相关的代码通常是难以阅读的，在软件工程中，代码可读性也是一个非常重要的指标。
- Go语言作为一门静态语言，编码过程中，编译器能提前发现一些类型错误，但是对于反射代码是无能为力的。所以包含反射相关的代码，很可能会运行很久才会出错，这时候经常是直接panic，可能会造成严重的后果。
- 反射对性能影响还是比较大的，比正常代码运行速度慢一到两个数量级。所以，对于一个项目中处于运行效率关键位置的代码，尽量避免使用反射特性。

### 2. 反射的作用

#### 2.1 在编写不定传参类型函数的时，或传入类型过多时

**典型应用为对象关系映射**

```go
type User struct {
  gorm.Model
  Name         string
  Age          sql.NullInt64
  Birthday     *time.Time
  Email        string  `gorm:"type:varchar(100);unique_index"`
  Role         string  `gorm:"size:255"` // set field size to 255
  MemberNumber *string `gorm:"unique;not null"` // set member number to unique and not null
  Num          int     `gorm:"AUTO_INCREMENT"` // set num to auto incrementable
  Address      string  `gorm:"index:addr"` // create index with name `addr` for address
  IgnoreMe     int     `gorm:"-"` // ignore this field
}

var users []User
db.Find(&users)
```

#### 2.2 不确定调用哪个函数，需要根据某些条件来动态执行

```go
func bridge(funcPtr interface{}, args ...interface{})
```

第一个参数funcPtr以接口的形式传入函数指针，函数参数args以可变参数的形式传入，bridge函数中可以用反射来动态执行funcPtr函数。

### 3. 反射的实现

Go的反射基础是接口和类型系统，Go的反射机制是通过接口来进行的。

Go语言在reflect包里定义了各种类型，实现了反射的各种函数，通过它们可以在运行时检测类型的信息、改变类型的值。

#### 3.1 反射三定律

根据Go官方关于反射的博客，反射有三大定律：

> 1. Reflection goes from interface value to reflection object.

> 2. Reflection goes from reflection object to interface value.

> 3. To modify a reflection object, the value must be settable.

- 反射可以将“接口类型变量”转换为“反射类型对象”。

  反射提供一种机制，允许程序在运行时访问接口内的数据。首先介绍下reflect包中的两个方法reflect.Value和reflect.TypeO

  ```go
  package main
  
  import (
  	"fmt"
  	"reflect"
  )
  
  func main() {
  	var Num float64 = 3.14
  
  	v := reflect.ValueOf(Num)
  	t := reflect.TypeOf(Num)
  
  	fmt.Println("Reflect : Num.Value = ", v)
  	fmt.Println("Reflect : Num.Type  = ", t)
  }
  ```

  返回

  ```shell
  Reflect : Num.Value =  3.14
  Reflect : Num.Type  =  float64
  ```

  上面的例子通过reflect.Value和reflect.TypeOf将接口类型变量分别转换为反射类型对象v和t，v是Num的值，t也是Num的类型。

  先来看一下reflectValueOf和reflectTypeOf的函数签名

  ```go
  func TypeOf(i interface{}) Type
  ```

  ```go
  func (v Value) Interface() (i interface{})
  ```

  两个方法的参数类型都是空接口

  在整个过程中，当我们调用reflect.TypeOf(x)的时候，Num会被存储在这个空接口中，然后reflect.TypeOf在对空接口进行拆解，将接口类型变量转换为反射类型变量。

- 反射可以将“反射类型变量”转换为”接口类型变量“。

  定律二是定律一的反过程。

  根据一个reflect.Value类型的变量，我们可以使用Interface方法恢复其接口类型的值。

  ```go
  package main
  import{
      "fmt",
      "reflect"
  }
  func main() {
      var Num = 3.14
      v := reflect.ValueOf(Num)
      t := reflect.TypeOf(Num)
      fmt.Print(v)
      fmt.Print(t)
      
      origin := v.Interface().(float64)
      fmt.Print(origin)
  }
  ```

  返回

  ```shell
  3.14
  float64
  3.14
  ```

- 如果要修改“反射类型对象”，其值必须是“可写的”。

  运行下程序：

  ```go
  package main
  
  import (
  	"reflect"
  )
  func main(){
  	var Num float64 = 3.14
  	v := reflect.ValueOf(Num)
  	v.SetFloat(6.18)
  }
  ```

  出现了panic

  ```shell
  panic: reflect: reflect.Value.SetFloat using unaddressable value
  
  goroutine 1 [running]:
  reflect.flag.mustBeAssignableSlow(0x8e)
  	C:/Go/src/reflect/value.go:259 +0x146
  reflect.flag.mustBeAssignable(...)
  	C:/Go/src/reflect/value.go:246
  reflect.Value.SetFloat(0x5cbc20, 0xc00009c000, 0x8e, 0x4018b851eb851eb8)
  	C:/Go/src/reflect/value.go:1609 +0x3e
  main.main()
  	G:/go-project/go-learning/11-反射机制.go:9 +0xba
  ```

  因为反射对象v包含的是副本值，所以无法修改。

  我们可以通过CanSet函数来判断反射对象是否可以修改，如下：

  ```go
  package main
  
  import (
      "fmt"
      "reflect"
  )
  func main(){
  	var Num float64 = 3.14
  	v := reflect.ValueOf(Num)
     fmt.Println("v的可写性：", v.CanSet())
  }
  ```

  输出：

  ```shell
  v的可写性： false
  ```

#### 小结

1. 反射对象包含了接口变量中存储的值以及类型。
2. 如果反射对象中包含的值是原始值，那么可以通过反射对象修改原始值。
3. 如果反射对象中包含的值不是原始值（反射对象包含的是副本值或指向原始值的地址），则该反射对象不可以修改。

### 4.反射的实践

- 通过反射修改内容

  ```go
  var f float64 = 3.41
  fmt.Println(f)
  p := reflect.ValueOf(&f)
  v := p.Elem()
  v.SetFloat(6.18)
  fmt.Println(f)
  ```

  reflect.Elem()方法获取这个指针指向的元素类型。这个获取过程被称为取元素，等效于对指针类型变量做一个*的操作。

- 通过反射调用方法

  ```go
  package main
  
  import (
      "fmt"
      "reflect"
  )
  
  func hello() {
      fmt.Println("Hello World!")
  }
  
  func main() {
      h1 := hello
      fv := reflect.ValueOf(h1)
      fv.Call(nil)
  }
  ```

反射会使得代码执行效率变慢，原因有

1. 涉及到内存分配以及后续的垃圾回收；
2. reflect实现里面有大量的枚举，也就是for循环，比如类型之类的。

### 实践

```go
package main

import (
	"reflect"
	"fmt"
)

type Child struct {
	Name     string
	handsome bool
}

func main() {
	qcrao := Child{Name: "qcrao", handsome: true}

	v := reflect.ValueOf(&qcrao)

	f := v.Elem().FieldByName("Name")
	fmt.Println(f.String())

	f.SetString("stefno")
	fmt.Println(f.String())

	f = v.Elem().FieldByName("handsome")
	
	// 这一句会导致 panic，因为 handsome 字段未导出
	//f.SetBool(true)
	fmt.Println(f.Bool())
}
```

执行结果：

```shell
qcrao
stefno
true
```

本例中，handsome字段未导出，可以读取，但不能调用相关set方法，否则会panic。反射用起来一定要注意，调用类型不匹配的方法，会导致各种panic。

## 二、反射的应用

反射的实际应用非常广：IDE 中的代码自动补全功能、对象序列化（json 函数库）、fmt 相关函数的实现、ORM（全称是：Object Relational Mapping，对象关系映射）……

这里举 2 个例子：json 序列化和 DeepEqual 函数。

### 1. json 序列化

`json` 是一种独立于语言的数据格式。最早用于浏览器和服务器之间的实时无状态的数据交换，并由此发展起来。

Go 语言中，主要提供 2 个函数用于序列化和反序列化：

```golang
func Marshal(v interface{}) ([]byte, error)
func Unmarshal(data []byte, v interface{}) error
```

两个函数的参数都包含 `interface`，具体实现的时候，都会用到反射相关的特性。

对于序列化和反序列化函数，均需要知道参数的所有字段，包括字段类型和值，再调用相关的 get 函数或者 set 函数进行实际的操作。

### 2. DeepEqual 的作用及原理

在测试函数中，经常会需要这样的函数：判断两个变量的实际内容完全一致。

例如：如何判断两个 slice 所有的元素完全相同；如何判断两个 map 的 key 和 value 完全相同等等。

上述问题，可以通过 `DeepEqual` 函数实现。

```golang
func DeepEqual(x, y interface{}) bool
```

`DeepEqual` 函数的参数是两个 `interface`，实际上也就是可以输入任意类型，输出 true 或者 flase 表示输入的两个变量是否是“深度”相等。

先明白一点，如果是不同的类型，即使是底层类型相同，相应的值也相同，那么两者也不是“深度”相等。

```golang
type MyInt int
type YourInt int

func main() {
	m := MyInt(1)
	y := YourInt(1)

	fmt.Println(reflect.DeepEqual(m, y)) // false
}
```

上面的代码中，m, y 底层都是 int，而且值都是 1，但是两者静态类型不同，前者是 `MyInt`，后者是 `YourInt`，因此两者不是“深度”相等。

在源码里，有对 DeepEqual 函数的非常清楚地注释，列举了不同类型，DeepEqual 的比较情形，这里做一个总结：

| 类型                                  | 深度相等情形                                                 |
| ------------------------------------- | ------------------------------------------------------------ |
| Array                                 | 相同索引处的元素“深度”相等                                   |
| Struct                                | 相应字段，包含导出和不导出，“深度”相等                       |
| Func                                  | 只有两者都是 nil 时                                          |
| Interface                             | 两者存储的具体值“深度”相等                                   |
| Map                                   | 1、都为 nil；2、非空、长度相等，指向同一个 map 实体对象，或者相应的 key 指向的 value “深度”相等 |
| Pointer                               | 1、使用 == 比较的结果相等；2、指向的实体“深度”相等           |
| Slice                                 | 1、都为 nil；2、非空、长度相等，首元素指向同一个底层数组的相同元素，即 &x[0] == &y[0] 或者 相同索引处的元素“深度”相等 |
| numbers, bools, strings, and channels | 使用 == 比较的结果为真                                       |

一般情况下，DeepEqual 的实现只需要递归地调用 == 就可以比较两个变量是否是真的“深度”相等。

但是，有一些异常情况：比如 func 类型是不可比较的类型，只有在两个 func 类型都是 nil 的情况下，才是“深度”相等；float 类型，由于精度的原因，也是不能使用 == 比较的；包含 func 类型或者 float 类型的 struct， interface， array 等。

对于指针而言，当两个值相等的指针就是“深度”相等，因为两者指向的内容是相等的，即使两者指向的是 func 类型或者 float 类型，这种情况下不关心指针所指向的内容。

同样，对于指向相同 slice， map 的两个变量也是“深度”相等的，不关心 slice， map 具体的内容。

对于“有环”的类型，比如循环链表，比较两者是否“深度”相等的过程中，需要对已比较的内容作一个标记，一旦发现两个指针之前比较过，立即停止比较，并判定二者是深度相等的。这样做的原因是，及时停止比较，避免陷入无限循环。

来看源码：

```golang
func DeepEqual(x, y interface{}) bool {
	if x == nil || y == nil {
		return x == y
	}
	v1 := ValueOf(x)
	v2 := ValueOf(y)
	if v1.Type() != v2.Type() {
		return false
	}
	return deepValueEqual(v1, v2, make(map[visit]bool), 0)
}
```

首先查看两者是否有一个是 nil 的情况，这种情况下，只有两者都是 nil，函数才会返回 true。

接着，使用反射，获取x，y 的反射对象，并且立即比较两者的类型，根据前面的内容，这里实际上是动态类型，如果类型不同，直接返回 false。

最后，最核心的内容在子函数 `deepValueEqual` 中。

代码比较长，思路却比较简单清晰：核心是一个 switch 语句，识别输入参数的不同类型，分别递归调用 deepValueEqual 函数，一直递归到最基本的数据类型，比较 int，string 等可以直接得出 true 或者 false，再一层层地返回，最终得到“深度”相等的比较结果。

实际上，各种类型的比较套路比较相似，这里就直接节选一个稍微复杂一点的 `map` 类型的比较：

```golang
// deepValueEqual 函数
// ……

case Map:
	if v1.IsNil() != v2.IsNil() {
		return false
	}
	if v1.Len() != v2.Len() {
		return false
	}
	if v1.Pointer() == v2.Pointer() {
		return true
	}
	for _, k := range v1.MapKeys() {
		val1 := v1.MapIndex(k)
		val2 := v2.MapIndex(k)
		if !val1.IsValid() || !val2.IsValid() || !deepValueEqual(v1.MapIndex(k), v2.MapIndex(k), visited, depth+1) {
			return false
		}
	}
	return true
	
// ……	
```

和前文总结的表格里，比较 map 是否相等的思路比较一致，也不需要多说什么。说明一点，`visited` 是一个 map，记录递归过程中，比较过的“对”：

```golang
type visit struct {
	a1  unsafe.Pointer
	a2  unsafe.Pointer
	typ Type
}

map[visit]bool
```

比较过程中，一旦发现比较的“对”，已经在 map 里出现过的话，直接判定“深度”比较结果的是 `true`。

## 参考

1. [深度解密Go语言之反射](https://www.cnblogs.com/qcrao-2018/p/10822655.html)
2. [Go语言圣经](http://shouce.jb51.net/gopl-zh/ch1/ch1-01.html)

