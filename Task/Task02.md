# Task02：数据类型、关键字、标识符

## 1. 数据类型

### 1.1 按类别

- 布尔型：只可以是常量 true 或者 false。

```go
eg:
var b bool = true
```

- 数字类型：整型和浮点型。

- 位的运算采用补码字符串类型：字符串就是一串固定长度的字符连接起来的字符序列，Go 的字符串是由单个字节连接起来。

- Go 语言的字符串的字节使用 UTF-8 编码标识 Unicode 文本

- 复数：complex128（64 位实数和虚数）和 complex64（32 位实数和虚数），其中 complex128 为复数的默认类型。
  注：

1. 复数的值由三部分组成 RE + IMi，其中 RE 是实数部分，IM 是虚数部分，RE 和 IM 均为 float 类型，而最后的 i 是虚数单位。

```go
var name complex128 = complex(x, y)
或者
z := complex(x, y)
x = real(z)
y = imag(z)
```

    2. 复数也可以用==和!=进行相等比较，只有两个复数的实部和虚部都相等的时候它们才是相等的

### 1.2 派生类型

- 指针类型（Pointer）
- 数组类型
- 结构化类型(struct)
- Channel 类型
- 函数类型
- 切片类型
- 接口类型（interface）
- Map 类型

### 1.3 基于架构

1. 整型，同时提供了四种有符号整型，分别对应8、16、32、64bit（二进制）的有符号整数，与此对应四种无符号的整数类型

- Uint8无符号 8 位整型 (0 到 255)
- Uint16
- Uint32
- Uint64
- int8
- int16
- int32
- int64

2. 浮点型：

- float32
- float64
- complex64(实数虚数)
- complex128

3. 其他：

- byte
- rune
- uint
- int
- uintptr(无符号整型，存放一个指针)

注：

1. 表示 Unicode 字符的 rune 类型和 int32 类型是等价的，通常用于表示一个 Unicode 码点，是等价的。
2. byte 和 uint8 也是等价类型，byte 类型一般用于强调数值是一个原始的数据而不是一个小的整数。
3. 无符号的整数类型 uintptr，它没有指定具体的 bit 大小但是足以容纳指针。只有在底层编程时才需要，特别是Go语言和C语言函数库或操作系统接口相交互的地方。
4. 有符号整数采用 2 的补码形式表示，也就是最高 bit 位用来表示符号位，一个 n-bit 的有符号数的取值范围是从 -2(n-1) 到 2(n-1)-1。无符号整数的所有 bit 位都用于表示非负数，取值范围是 0 到 2n-1。
5. 常量 math.MaxFloat32 表示 float32 能取到的最大数值，大约是 3.4e38。
6. 常量 math.MaxFloat64 表示 float64 能取到的最大数值，大约是 1.8e308。
7. float32 和 float64 能表示的最小值分别为 1.4e-45 和 4.9e-324。
8. 浮点数在声明的时候可以只写整数部分或者小数部分。

```go
const e = .71828 // 0.71828
const f = 1.     // 1
```

9. 很小或很大的数最好用科学计数法书写，通过 e 或 E 来指定指数部分

```go
const Avogadro = 6.02214129e23  // 阿伏伽德罗常数
const Planck   = 6.62606957e-34 // 普朗克常数
```

### 2. 关键字

#### 2.1 25个关键字或保留字

break default func interface select
case defer go map struct
chan else goto package switch
const fallthrough if range type
continue for import return var

#### 2.2 36 个预定义标识符

append bool byte cap close complex complex64 complex128 uint16
copy false float32 float64 imag int int8 int16 uint32
int32 int64 iota len make new nil panic uint64
print println real recover string true uint uint8 uintptr

#### 2.3 知识点

- 程序一般由关键字、常量、变量、运算符、类型和函数组成。
- 程序中可能会使用到这些分隔符：括号 ()，中括号 [] 和大括号 {}。
- 程序中可能会使用到这些标点符号：.、,、;、: 和 …。

### 3. 标识符

标识符用来命名变量、类型等程序实体。一个标识符实际上就是一个或是多个字母(A~ Z和a~ z)数字(0~9)、下划线“_”组成的序列，但是第一个字符必须是字母或下划线而不能是数字。

## 实战代码

```go
/*变量*/
	//普通赋值
	var num int = 1
	fmt.Println(num) //1
	//平均赋值
	var num1,num2 = 2,3
	fmt.Println(num1,num2) //2 3
	//
	var (
		num3 int= 3
		num4 int = 4
	)
	fmt.Println(num3,num4) //3 4
/*整数类型*/
	var num8 int8= 027  // 23
	fmt.Println(num8) //按十进制进行输出
	var num16 = 0x39 //57
	fmt.Println(num16) //按十进制进行输出
/*浮点数*/
	var floatnum float32 = 58.3e-2
	fmt.Println(floatnum) //0.583
	num = int(floatnum) //float32转int64
	fmt.Println(num) //0

/*复数类型测试*/
	//var x float32= 10.0
	//var y float32= 13.0
	// cannot use complex(x, y) (type complex64) as type complex128 in assignment
	//64位的复数，需要64位的float，不能是其它类型
	var x float64= 10.0
	var y float64= 13.0
	var name complex128 = complex(x, y)

	fmt.Println(name) //(10+13i)
	var z complex128 = name
	fmt.Println(z) //(10+13i)
	fmt.Println(real(name)) //10
	fmt.Println(imag(name)) //13
	fmt.Println(name == z) //true
/*别名类型byte和rune*/
	//byte == uint8
	//var b1 byte = 256 //constant 256 overflows byte
	var b1 byte = 255 //constant 256 overflows byte
	fmt.Println(b1) //255

	//rune == int32   与Unicode编码对应
	//var r1 rune = math.MaxInt32 + 1 //constant 2147483648 overflows rune
	var r1 rune = math.MaxInt32 //2147483647
	fmt.Println(r1)
	var r2 rune = 'A'
	fmt.Println(r2) //65   输出了A字符代表的Unicode编码号

/*字符串类型*/
	var str1 string = `hello\nabc`  //原生表示法   反引号  在左上角而非 单引号
	var str2 string = "world\nabc"  //解释性表示法  会对转义字符进行解释
	fmt.Println(str1+str2)  //hello\nabcworld
							//abc

/*数组类型*/
	//长度确定
	type nums1 [3]int //只是声明一个[3]int的数组
	//fmt.Println(nums1) //Type nums1 is not an expression
	var nums2 = [3]int{1,2,3}  //起别名     PS:起别名时无需指定类型
	var nums3 = [...]int{1,2,3,4,5}  //起别名
	fmt.Println(nums2)  //[1 2 3]
	fmt.Println(nums3)  //[1 2 3 4 5]
	fmt.Println(nums3[2])  //3
	//修改nums3中的值
	nums3[2] = 9
	fmt.Println(nums3) //[1 2 9 4 5]
	var length = len(nums3)   //PS:起别名时无需指定类型
	fmt.Println(length) //5

/*切片类型*/
	//属于引用类型，长度不确定，底层用数组
	type sliceInt []int //声明的作用？
	s := sliceInt{1, 2, 3} //[1 2 3]   :=是什么操作？
	fmt.Println(s)
	var sliceInt1 = []int{4,5,6,7,8,9}  //[4 5 6 7 8 9]
	fmt.Println(sliceInt1)
	//切片
	var slice1 = sliceInt1[1:4]  //[5 6 7]  前闭后开，类似于python
	fmt.Println(slice1)
	//切片的长度与容量：一个切片值的容量即为它的第一个元素值在其底层数组中的索引值与该数组长度的差值的绝对值
	fmt.Println(len(s)) //3
	fmt.Println(len(sliceInt1)) //6
	fmt.Println(len(slice1)) //3
	fmt.Println(cap(s)) //3
	fmt.Println(cap(sliceInt1)) //6
	fmt.Println(cap(slice1)) //5     [5,6,7,8,9]
	//切片值的改变会影响切片原始的值
	slice1[0] = 100
	fmt.Println(sliceInt1)  //[4 100 6 7 8 9]
	//通过切片的第三个值进行限制切片的访问
	var slice2 = sliceInt1[0:2:3]  //3
	fmt.Println(cap(slice2))
	//延展切片
	fmt.Println(slice1[:5])  //原本slice1的值为[5 6 7] 现在输出  [100 6 7 8 9]  意为延展其长度为5，数据参照slice1从哪里切过来的，就从哪里可以观察到   这个5是cap(slice1)
	//在切片中加入第三个索引后， var slice1 = numbers3[1:4:4] 将无法通过slice1访问到number3的值中的第五个元素。
	//使用内建函数append可以不受限制的扩展,并返回一个新的切片值,与
	fmt.Println(slice1) //[100 6 7]
	var slice3 = append(slice1, 40,41,42)
	fmt.Println(sliceInt1) // [4 100 6 7 8 9]  与最初是的切片值的来源将没有关系
	fmt.Println(slice3) //[100 6 7 40 41 42]


//copy函数会直接对其第一个参数值进行对应位置的修改
var slice4 = []int{0, 0, 0, 0, 0, 0, 0, 0}

fmt.Println(copy(slice4,slice1)) //3   意为替换了3个
fmt.Println(slice4)  //[100 6 7 0 0 0 0 0]
```