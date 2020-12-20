# Task07:函数

## 函数

### 1. 函数定义

在go语言中函数定义格式如下：

```go
func 函数名(参数列表)返回值类型{
    //函数体
}

func functionName([parameter list]) [returnTypes]{
   //body
}
```

- 函数由func关键字进行声明。
- functionName：代表函数名。
- parameter list：代表参数列表，函数的参数是可选的，可以包含参数也可以不包含参数。
- returnTypes：返回值类型，返回值是可选的，可以有返回值，也可以没有返回值。
- body：用于写函数的具体逻辑

例1:

下面的函数是用于求两个数的和

```go
func GetSum(num1 int, num2 int) int {
	result := num1 + num2
	return result
}
```

这个函数传递了两个参数，分别为`num1`与`num2`，并且他们都为`int`类型，将相加后的结果进行返回。

上面这个函数还可以这样定义

```go
func GetSum1(num1, num2 int) int {
	result := num1 + num2
	return result
}
```

当num1和num2是相同类型的时候我们可以省略掉前面的类型，go编译器会自动进行推断。

### 2. 值传递与引用传递

因为在go语言中存在值类型与引用类型，所以在函数参数进行传递时也要注意这个问题。

- 值传递是指在函数调用过程中将实参拷贝一份到函数中，这样在函数中如果对参数进行修改，将不会影响到实参。
- 引用传递是指在函数调用过程中将实参的地址传递到函数中，那么在函数中对参数所进行的修改，将影响到实参。

如果想要函数可以直接修改参数的值，那么我们可以用指针传递（引用传递），将变量的地址作为参数传递到函数中。下面的这个例子为大家演示了以上的几种情况。

例2：

```go
func paramFunc(a int, b *int, c []int) {
	a = 100
	*b = 100
	c[1] = 100

	fmt.Println("paramFunc:")
	fmt.Println(a)
	fmt.Println(*b)  //改变了
	fmt.Println(c)  //改变了
}

func main() {
	a := 1
	b := 1
	c := []int{1, 2, 3}
	paramFunc(a, &b, c)

	fmt.Println("main:")
	fmt.Println(a)
	fmt.Println(b) 
	fmt.Println(c)
}
```

程序输出如下

```go
paramFunc:
100
100
[1 100 3]
main:
1
100
[1 100 3]
```

### 3. 变长参数

在go语言中也支持变长参数，但需要注意的是变长参数必须放在函数参数的最后一个，否则会报错。

下面这段代码演示了如何使用变长参数

例3：

```go
func main() {
	slice := []int{7, 9, 3, 5, 1}
	x := min(slice...)
	fmt.Printf("The minimum is: %d", x)
}

func min(s ...int) int {
	if len(s) == 0 {
		return 0
	}
	min := s[0]
	for _, v := range s {
		if v < min {
			min = v
		}
	}
	return min
}
```

当然上面这段代码直接将切片作为参数也能实现同样的效果，但是变长参数更多的是为了参数不确定的情况，例如fmt包中的Printf函数设计如下：

```go
func Printf(format string, a ...interface{}) (n int, err error) {
	return Fprintf(os.Stdout, format, a...)
}
```

上面这段代码暂时看不懂也没关系，但是只需要记住，当你想传递给函数的参数不能确定有多少时可以使用变长参数。

### 4. 多返回值

go语言中函数还支持一个特性那就是：多返回值。通过返回结果与一个错误值，这样可以使函数的调用者很方便的知道函数是否执行成功，这样的模式也被称为command,ok模式，在我们未来的程序设计中也推荐大家使用这种方式。下面这段代码显示了如何操作多返回值。

例4:

```go
func div(a, b float64) (float64, error) {
	if b == 0 {
		return 0, errors.New("The divisor cannot be zero.")
	}
	return a / b, nil
}

func main() {
	result, err := div(1, 2)
	if err != nil {
		fmt.Printf("error: %v", err)
		return
	}
	fmt.Println("result: ", result)
}
```

也可以下面这种模式

```go
func main() {
	if result, err := div(1, 2); err != nil {
		fmt.Printf("error: %v", err)
		return
	} else {
		fmt.Println("result: ", result)
	}
}
```

注：多返回值需要使用`()`进行标记。

### 5. 命名返回值

除了上面支持的多返回值，在go语言中还可以给返回值命名，当需要返回的时候，我们只需要一条简单的不带参数的return语句。我们将上面那个除法的函数修改一下

例5:

```go
func div(a, b float64) (result float64, err error) {
	if b == 0 {
		return 0, errors.New("被除数不能等于0")
	}
	result = a / b
	return
}
```

注：即使只有一个命名返回值，也需要使用`()`括起来。

### 6. 匿名函数

匿名函数如其名字一样，是一个没有名字的函数，除了没有名字外其他地方与正常函数相同。匿名函数可以直接调用，保存到变量，作为参数或者返回值。

例6:

```go
func main() {
	f := func() string {
		return "hello world"
	}
	fmt.Println(f())
}
```

### 7. 闭包

闭包可以解释为**一个函数与这个函数外部变量的一个封装**。粗略的可以理解为一个类，类里面有变量和方法，**其中闭包所包含的外部变量对应着类中的静态变量。** 为什么这么理解，首先让我们来看一个例子。

例7:

```go
func add() func(int) int {
	n := 10
	str := "string"
	return func(x int) int {
		n = n + x
		str += strconv.Itoa(x)
		fmt.Print(str, " ")
		return n
	}
}

func main() {
	f := add()
	fmt.Println(f(1))
	fmt.Println(f(2))
	fmt.Println(f(3))

	f = add()
	fmt.Println(f(1))
	fmt.Println(f(2))
	fmt.Println(f(3))
}
```

程序输出结果如下：

```go
string1 11
string12 13
string123 16
string1 11
string12 13
string123 16
```

如果不了解的闭包肯定会觉得很奇怪，为什么会输出这样的结果。这就要用到我最开始的解释。**闭包就是一个函数和一个函数外的变量的封装，而且这个变量就对应着类中的静态变量。** 这样就可以将这个程序的输出结果解释的通了。

- 最开始我们先声明一个函数`add`，在函数体内返回一个匿名函数
- 其中的`n`,`str`与下面的匿名函数构成了整个的闭包，`n`与`str`就像类中的静态变量只会初始化一次，所以说尽管后面多次调用这个整体函数，里面都不会再重新初始化了
- 而且对于外部变量的操作是累加的，这与类中的静态变量也是一致的

在go语言学习笔记中，雨痕提到在汇编代码中，闭包返回的不仅仅是匿名函数，还包括所引用的环境变量指针，这与我们之前的解释也是类似的，闭包通过操作指针来调用对应的变量。

小问题：

尝试一下如何通过闭包来实现斐波那契数列。

## 实战

### 1. 参数与返回值

return可以有参数，也可以没有参数，这些返回值可以有名称，也可以没有名称。Go中的函数可以有多个返回值。

- (1).当返回值有多个时，这些返回值必须使用括号包围，逗号分隔
- (2).return关键字中指定了参数时，返回值可以不用名称。如果return省略参数，则返回值部分必须带名称
- (3).当返回值有名称时，必须使用括号包围，逗号分隔，即使只有一个返回值
- (4).但即使返回值命名了，return中也可以强制指定其它返回值的名称，也就是说return的优先级更高
- (5).命名的返回值是预先声明好的，在函数内部可以直接使用，无需再次声明。命名返回值的名称不能和函数参数名称相同，否则报错提示变量重复定义
- (6).return中可以有表达式，但不能出现赋值表达式，这和其它语言可能有所不同。例如`return a+b`是正确的，但`return c=a+b`是错误的

```go
// 单个返回值
func func_a() int{
    return a
}

// 只要命名了返回值，必须括号包围
func func_b() (a int){
    // 变量a int已存在，无需再次声明
    a = 10
    return
    // 等价于：return a
}

// 多个返回值，且在return中指定返回的内容
func func_c() (int,int){
    return a,b
}

// 多个返回值
func func_d() (a,b int){
    return
    // 等价于：return a,b
}

// return覆盖命名返回值
func func_e() (a,b int){
    return x,y
}
```

### 2. 按值传参

Go中是通过传值的方式传参的，意味着传递给函数的是拷贝后的副本，所以函数内部访问、修改的也是这个副本。

```go
a,b := 10,20
min(a,b)
func min(x,y int) int{}
```

上面调用min()时，是将a和b的值拷贝一份，然后将拷贝的副本赋值给变量x,y的，所以min()函数内部，访问、修改的一直是a、b的副本，和原始的数据对象a、b没有任何关系。

如果想要修改外部数据(即上面的a、b)，需要传递指针。

例如，下面两个函数，`func_value()`是传值函数，`func_ptr()`是传指针函数，它们都修改同一个变量的值。

```go
package main

import "fmt"

func main() {
	a := 10
	func_value(a)
	fmt.Println(a)    // 输出的值仍然是10
    
	b := &a
	func_ptr(b)
	fmt.Println(*b)   // 输出修改后的值：11
}

func func_value(x int) int{
	x = x + 1
	return x
}

func func_ptr(x *int) int{
	*x = *x + 1
	return *x
}
```

### 3.斐波那契数列

```go
package main

import "fmt"

// 返回一个“返回int的函数”
func fibonacci() func() int {
    i := -1
	j := 0
	return func() int {
	    if i == -1 {
		    i += 1
		    return 0
	    } else if j == 0 {
		    j += 1
			return 1
		} else {
			s := i + j
		    i, j = j, s
		    return s
		}
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```

