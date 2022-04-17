---
title: go
date: 2022-04-17 13:28:23
tags: Go learn
category: GO
---

# GO

###编译型语言

*特点

```sh
1.语法简洁
2.开发效率高
3.执行性能好
```



## 编译

### go build

1.在项目目录下执行go build

2.在其他路径下执行go build，需要在后面加上项目的路径（项目路径从GOPATH/src后开始写起，编译之后的可执行文件就保存在当前目录下）

3.go build -o hello.exe



### go run

像执行脚本文件一样执行Go代码



### go install

go install 分为两步:
1.先编译得到一个可执行文件
2.将执行文件拷贝到`GOPATH/bin`



### 跨平台编译

```bash
SET CGO_ENABLED=0  // 禁用CGO
SET GOOS=windows  // 目标平台是windows
SET GOARCH=amd64  // 目标处理器架构是amd64

#Windows编译Mac可执行文件
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build

#PowerShell
##使用环境变量的语法
$ENV:CGO_ENABLED=0
$ENV:GOOS="darwin"
$ENV:GOARCH="amd64"
go build

#Linux编译Windows可执行文件
CGO_ENABLED=0 
GOOS=windows 
GOARCH=amd64 
go build

#Linux编译Mac可执行文件
CGO_ENABLED=0 
GOOS=darwin 
GOARCH=amd64 
go build

#Mac编译Linux可执行文件
export CGO_ENABLED=0 
export GOOS=linux 
export GOARCH=amd64 
go build

#Mac编译Windows可执行文件
export CGO_ENABLED=0 
export GOOS=windows 
export GOARCH=amd64 
go build
```



### go语言中有25个关键字

```go
    break        default      func         interface    select
    case         defer        go           map          struct
    chan         else         goto         package      switch
    const        fallthrough  if           range        type
    continue     for          import       return       var
```



```text
注意事项：

1. 函数外的每个语句都必须以关键字开始（var、const、func等）
2. `:=`不能使用在函数外。
3. `_`多用于占位，表示忽略值。
```



## 变量

```text
var 变量声明

匿名变量  在使用多重赋值时，如果想要忽略某个值，可以使用匿名变量（anonymous variable）。 匿名变量用一个下划线_表示,匿名变量不占用命名空间，不会分配内存，所以匿名变量之间不存在重复声明。

const	相对于变量，常量是恒定不变的值，多用于定义程序运行期间不会改变的那些值。 常量的声明和变量声明非常类似，只是把var换成了const，常量在定义的时候必须赋值。

iost 	是go语言的常量计数器，只能在常量的表达式中使用。
```

| 转义符 |                含义                |
| :----: | :--------------------------------: |
|  `\r`  |         回车符（返回行首）         |
|  `\n`  | 换行符（直接跳到下一行的同列位置） |
|  `\t`  |               制表符               |
|  `\'`  |               单引号               |
|  `\"`  |               双引号               |
|  `\\`  |               反斜杠               |

| 格式化指令 | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| %%         | %字面量                                                      |
| %b         | 一个二进制整数，将一个整数格式转化为二进制的表达方式         |
| %c         | 一个Unicode的字符                                            |
| %d         | 十进制整数                                                   |
| %o         | 八进制整数                                                   |
| %x         | 小写的十六进制数值                                           |
| %X         | 大写的十六进制数值                                           |
| %U         | 一个Unicode表示法表示的整型码值                              |
| %s         | 输出以原生的UTF8字节表示的字符，如果console不支持utf8编码，则会乱码 |
| %t         | 以true或者false的方式输出布尔值                              |
| %v         | 使用默认格式输出值，或者如果方法存在，则使用类性值的String()方法输出自定义值 |
| %T         | 输出值的类型                                                 |

**var**（声明变量）

```go
package main

import "fmt"

//声明变量
// var name string
// var age int
// yes  bool

// 批量声明
var (
	name string // “”
	age  int    // 0
	yes  bool   // false
)

func main() {
	name = "zzz"
	age = 18
	yes = true
	// var zyl int
	// Go语言中非全局变量声明后必须使用,不使用就编译不过去
	fmt.Print(yes)                //在终端输出要打印的内容
	fmt.Printf("name:%s\n", name) // %s是占位符 使用name变量值去替换%s
	fmt.Println(age)              // 打印完指定的内容之后会在后面加一个换行符

	// 声明变量同时赋值
	var zyl string = "test"
	fmt.Println(zyl)

	// 类型推倒（根据值判断该变量是什么类型）
	var age1 = "20"
	fmt.Println(age1)

	// 简短变量声明,只能在函数中声明
	s1 := "哈哈哈"
	fmt.Println(s1)
	// 同一个作用域({})中不能重复声明同名变量
}
```

## if 和 for

```go
package main

import "fmt"

func main() {
	score := 65
	if score >= 90 {
		fmt.Println("A")
	} else if score > 75 {
		fmt.Println("B")
	} else {
		fmt.Println("C")
	}
}

func demo1() {
	if score := 65; score >= 90 {
		fmt.Println("A")
	} else if score > 75 {
		fmt.Println("B")
	} else {
		fmt.Println("C")
	}
}

func demo2() {
	for i := 0; i < 10; i++ {
		fmt.Println(i)
	}
}

func demo3() {
	i := 0
	for ; i < 10; i++ {
		fmt.Println(i)
	}
}

// 无限循环
// for {
//     循环体语句
// }
```

for range：

```go
package main

import (
	"fmt"
)

func main() {
	for i := 1; i < 10; i++ {
		fmt.Println(i)
	}
}

func main() {
	s := "hello 世界！"
	for i, v := range s {
		fmt.Printf("%d %c\n", i, v)
	}
}
//九九乘法表
func main() {
	for i := 1; i < 10; i++ {
		for j := 1; j <= i; j++ {
			q := i * j
			fmt.Printf("%d*%d=%d ", j, i, q)
		}
		fmt.Println()
	}
}
```

## const into

```go
package main

import "fmt"

// 常量定义
const pi = 3.1415926

// 多常量定义
const (
	n1 = 100
	n2
	n3
)

// iota是go语言的常量计数器，只能在常量的表达式中使用。
// iota在const关键字出现时将被重置为0。const中每新增一行常量声明将使iota计数一次(iota可理解为const语句块中的行索引)。 使用iota能简化定义，在定义枚举时很有用。
// 定义数量级
const (
	_  = iota
	KB = 1 << (10 * iota)
	MB = 1 << (10 * iota)
	GB = 1 << (10 * iota)
	TB = 1 << (10 * iota)
	PB = 1 << (10 * iota)
)

// iota
const (
	a1 = iota //0
	a2        //1
	a3        //2
	a4        //3
)

// _跳过某些值
const (
	b1 = iota //0
	b2        //1
	_
	b3 //3
)

// iota声明中间隔常量
const (
	c1 = iota //0
	c2 = 100  //100
	c3 = iota //2
	c4        //3
)
const n5 = iota //0

func main() {
	// fmt.Println(n1)
	// fmt.Println(n2)
	// fmt.Println(n3)

	fmt.Println("a1:", a1)
	fmt.Println("a2:", a2)
	fmt.Println("a3:", a3)
	fmt.Println("a4:", a4)

	fmt.Println("b1:", b1)
	fmt.Println("b2:", b2)
	fmt.Println("b3:", b3)

	fmt.Println("c1:", c1)
	fmt.Println("c2:", c2)
	fmt.Println("c3:", c3)
	fmt.Println("c4:", c4)
}
```

## switch和goto

```go
//switch
//简化大量的判断 （一个变量和具体的做比较）
func main() {
	var n = 5
	// if n == 1 {
	// 	fmt.Println("大拇指")
	// } else if n == 2 {
	// 	fmt.Println("食指")
	// } else if n == 3 {
	// 	fmt.Println("中指")
	// } else if n == 4 {
	// 	fmt.Println("无名指")
	// } else if n == 5 {
	// 	fmt.Println("小母指")
	// } else {
	// 	fmt.Println("判断错误！")
	// }
	// switch 简化
	switch n {
	case 1:
		fmt.Println("大拇指")
	case 2:
		fmt.Println("食指")
	case 3:
		fmt.Println("中指")
	case 4:
		fmt.Println("无名指")
	case 5:
		fmt.Println("小母指")
	default:
		fmt.Println("判断错误！")
	}
	// goto+label实现跳出多层for循环
	for i := 0; i < 10; i++ {
		for j := 'A'; j < 'Z'; j++ {
			if j == 'D' {
				goto xx //跳出循环到指定的标签
			}
			fmt.Printf("%d %c\n", i, j)
		}
	}
xx: //label标签
	fmt.Println("hello world")
}
```

## Array(数组)

数组是同一种数据类型元素的集合。 在Go语言中，数组从声明时就确定，使用时可以修改数组成员，但是数组大小不可变化。 基本语法：

```go
// 定义一个长度为3元素类型为int的数组a
var a [3]int
```

数组定义：

```go
var 数组变量名 [元素数量]T
```

**eg：**

```go
package main

import "fmt"

//数组
//存放元素的容器
//必须指定存放的元素的类型和容量（长度）
//数组的长度是数组类型的一部分

func main() {
	var a1 [3]bool
	var a2 [4]bool
	fmt.Printf("类型：a1:%T a2:%T\n", a1, a2)

	//数组初始化
	//数组不初始化默认元素为零（布偶值:false,整型和浮点型都是0,字符串:""）
	fmt.Println("默认值：", a1, a2)

	//初始化

	//方式一
	a1 = [3]bool{true, true, true}
	fmt.Println("方式一:", a1)

	//方式二
	// 根据初始值自动判断数组的长度是多少
	a10 := [...]int{0, 1, 2, 3, 4, 5, 6, 7}
	fmt.Println("方式二:", a10)

	//方式三
	//根据索引来初始化
	a3 := [5]int{1, 2}
	fmt.Println("方式三:", a3)
	a4 := [5]int{0: 1, 4: 2}
	fmt.Println("方式三:", a4)

	// //数组的遍历
	// citys := [...]string{"北京", "上海", "深圳"}
	// //1.根据索引遍历
	// for i := 0; i < len(citys); i++ {
	// 	fmt.Println(citys[i])
	// }
	// for i, v := range citys {
	// 	fmt.Println(i, v)
	// }

	//多维数组
	var a5 [3][2]int
	a5 = [3][2]int{
		{1, 2},
		{3, 4},
		{5, 6},
	}
	fmt.Println("多维数组：",a5)

	//多维数组遍历
	fmt.Println("多维数组遍历:")
	for _,v1 := range a5 {
		fmt.Println(v1)
		for _,v2 := range v1{
			fmt.Println(v2)
		}
	}
}
```

## slice(切片)

切片的本质

切片就是一个框，框住了一块连续的内存

切片属于引用类型，真正的数据都是保存在底层数组里的



判断一个切片是否为空,要使用len(s) == 0 来判断

```go
package main

import "fmt"

// 切片slice

func main() {
	// 切片定义
	var a1 []int    //定义一个存放int类型元素的切片
	var a2 []string //定义一个存放string类型元素的切片
	fmt.Println(a1, a2)
	fmt.Println(a1 == nil) // nil==null
	fmt.Println(a2 == nil)
	//初始化
	a1 = []int{1, 2, 3}
	a2 = []string{"哈哈", "嘿嘿", "哦噢"}
	fmt.Println(a1, a2)
	fmt.Println(a1 == nil)
	fmt.Println(a2 == nil)

	//长度和容量(len cap)
	fmt.Printf("len(a1):%d cap(a1):%d\n", len(a1), cap(a1))
	fmt.Printf("len(a2):%d cap(a2):%d\n", len(a2), cap(a2))
	//切片的容量是指从底层数组切片的第一个元素到最后一个元素
    
   	//make() 函数创造切片
	a3 := make([]int, 5, 10)
	fmt.Printf("a3=%v len:%d cap:%d\n", a3, len(a3), cap(a3))

	a4 := make([]int, 0, 10)
	fmt.Printf("a4=%v len:%d cap:%d\n", a4, len(a4), cap(a4))
}
```

### append()

```go
// append()为切片追加元素

a5 := []int{1, 2, 3, 4, 5}
// a5[5] = 6 //错误写法 索引越界
// append追加元素，原来底层数组放不下的时候，GO底层就会把底层数组换一个
a5 = append(a5, 6) // 调用append函数必须用原来的切片变量接收返回值
fmt.Printf("a5=%v len(a5)=%d cap(a5)=%d\n", a5, len(a5), cap(a5))

aa := []int{7, 8, 9 , 10}
a5 = append(a5, aa...) // ...表示拆开
fmt.Printf("a5=%v len(a5)=%d cap(a5)=%d\n", a5, len(a5), cap(a5))
```

切片扩容策略

```text
首先判断，如果新申请容量（cap）大于2倍的旧容量（old.cap），最终容量（newcap）就是新申请的容量（cap）。
否则判断，如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍，即（newcap=doublecap），
否则判断，如果旧切片长度大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始循环增加原来的1/4，即（newcap=old.cap,for {newcap += newcap/4}）直到最终容量（newcap）大于等于新申请的容量(cap)，即（newcap >= cap）
如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量（cap）。
注意：切片扩容还会根据切片中元素的类型不同而做不同的处理，比如int和string类型的处理方式就不一样。
```

```go
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {
	newcap = cap
} else {
	if old.len < 1024 {
		newcap = doublecap
	} else {
		// Check 0 < newcap to detect overflow
		// and prevent an infinite loop.
		for 0 < newcap && newcap < cap {
			newcap += newcap / 4
		}
		// Set newcap to the requested cap when
		// the newcap calculation overflowed.
		if newcap <= 0 {
			newcap = cap
		}
	}
}
```

### copy()

```
// copy()复制切片
a6 := a5
var a7 = make([]int,10,10) //需要计算好长度和容量
copy(a7, a5)
fmt.Println(a6,a7)
```

## 数组和切片的区别

```text
1.声明方式不同，数组声明需要指定长度（无论是显式指定还是编译器根据元素个数推断都是指定），切片则不用；

2.数组长度固定，无法改变，切片长度不固定，可以自动扩容；

3.数组是值类型，切片是引用类型。数组在作为参数传递时，是直接在内存中拷贝了一份数据进行传递，切片则传递的是引用地址，所以在方法中对数组进行修改不会影响原数组，而切片则会受影响；

4.切片可以用cap函数计算容量、len函数计算长度，数组只有len函数计算长度，换句话说，切片比数组多一个cap属性；

5.切片的底层是数组。

数组作为参数传递不会影响原数组的前提是：数组里面的元素不是指针，因为如果传递的数组里面的元素是指针，那么通过修改元素指针指向的值，也会影响到原数组中的数据。还有一种情况就是，将数组的指针作为参数传递，这样也会影响到原数组，但是这个实际上已经不是数组作为参数传递了，而是指针作为参数传递。
```

## 指针

```text
&：取地址
*：根据地址取值
```

## map

map是一种无序的基于`key-value`的数据结构，Go语言中的map是引用类型，必须初始化才能使用。

map类型的变量默认初始值为nil，需要使用make()函数来分配内存。语法为：

```go
make(map[KeyType]ValueType, [cap])
```

```go
package main

import (
	"fmt"
)

func main() {
	a := make(map[string]int, 5)
	a["哈哈"] = 100
	a["哦噢"] = 101
	a["阿阿"] = 102
	fmt.Println(a["哈哈"])
	fmt.Printf("type(a):%T\n", a)

	// map支持在声明的时候填充元素
	a1 := map[string]string{
		"username": "admin",
		"password": "123456",
	}
	fmt.Println(a1)

	//判断key是否存在
	a2 := make(map[string]string)
	a2["username"] = "admin"
	a2["password"] = "123456"
	v, ok := a2["password"]
	if ok {
		fmt.Println(v)
	} else {
		fmt.Println("error")
	}

	// map 遍历
	for k, v := range a2 {
		fmt.Println(k, v)
	}
	//遍历key
	for k := range a2 {
		fmt.Println(k)
	}
	//遍历values
	for _, v = range a2 {
		fmt.Println(v)
	}

	//delete 删除
	delete(a2, "password")
	fmt.Println(a2)

	//元素类型为map的切片
	var a8 = make([]map[string]string, 10, 10)
	//没有对内部的map做初始化
	a8[0] = make(map[string]string, 1)
	a8[0]["username"] = "admin"
	fmt.Println(a8)

	// 值为切片类型的map
	var a9 = make(map[string][]int,10)
	a9["username"] = []int{1,2,3}
	fmt.Println(a9)
}
```

## func(函数)

函数是组织好的、可重复使用的、用于执行指定任务的代码块。

```go
package main

import "fmt"

// 函数

// 函数是一段代码的封装

// 函数定义
func sum(x int, y int) (ret int) {
	return x + y
}

// 没有返回值
func f1(x int, y int) {
	fmt.Println(x + y)
}

// 没有参数没有返回值
func f2() {
	fmt.Println("f2")
}

// 没有参数但有返回值
func f3() int {
	return 3
}

// 返回值可以命名也可以不命名
// 命名的返回值相当于在函数中声明一个变量
func f4(x int, y int) (ret int) {
	ret = x + y
	return //命名返回值可以return后省略
}

// 多值返回
func f5() (int, string) {
	return 1, "哈哈"
}

// 参数的类型简写：当参数中连续的两个参数的类型一致时，我们可以将前面参数的类型简写
func f6(x, y int) int {
	return x + y
}

// 可变长参数
// 可变长参数必须放在函数参数的最后
func f7(x string, y ...int) {
	fmt.Println(x)
	fmt.Println(y)
}

// 匿名函数
var f8 = func(x, y int) {
	fmt.Println(x + y)
}

func main() {
	// a := sum(1, 2)
	// fmt.Println(a)

	// _, a1 := f5()
	// fmt.Println(a1)

	f8(10, 20)
	// 函数内部无法定义一个带名字的函数
	// func f9()  {
	// }
	f9 := func(x, y int) {
		fmt.Println(x + y)
	}
	f9(10, 10)

	// 如果只调用一次的函数，还可以简写成立即执行函数
	func(x, y int) {
		fmt.Println(x + y)
	}(10, 5)
}
```

## defer语句

Go语言中的defer语句会将其后面跟随的语句进行延迟处理。在defer归属的函数即将返回时，将延迟处理的语句按defer定义的逆序进行执行，也就是说，先被defer的语句最后被执行，最后被defer的语句，最先被执行。

函数中return语句在底层并不是原子操作，它分为给返回值赋值和RET指令两步。而defer语句执行的时机就在返回值赋值操作后，RET指令执行前。

```go
package main

import "fmt"

// defer

// defer多用于函数结束之前释放资源(文件句柄，数据库连接)
func deferDemo() {
	fmt.Println("start")
	defer fmt.Println("哈哈---1") // defer把它后面的语句延迟到函数即将返回时候在执行
	defer fmt.Println("哈哈--2")  // 一个函数中可以有多个defer语句
	defer fmt.Println("哈哈-3")   //多个defer语句按照先进后出的顺序延迟执行
	fmt.Println("end")
}

func f1() int {
	x := 5
	fmt.Println(&x)
	defer func() {
		x++ // 修改的不是x的返回值
		fmt.Println(x)
		fmt.Println(&x)
	}()
	fmt.Printf("------%d\n", x)
	return x
}

// 函数中return语句在底层并不是原子操作，它分为给返回值 赋值和RET指令两步。而defer语句执行的时机就在返回值赋值操作后，RET指令执行前

// 返回值开辟了一块新的内存空间0xc0000140d0，未命名。赋值为5
// 运行defer x++ x=6 ,x跟返回值已经没有关系了
// ruturn 是指向0xc0000140d0 这片区域的值

// 返回值开辟了一块新的内存空间0x----，赋值为x=5
// 运行defer x=6 。同时修改了指向0x----的值
// return 指向0x-----区域的值
func f2() (x int) {
	defer func() {
		x++
	}()
	return 5 // 返回值=x
}

func f3() (y int) {
	x := 5
	defer func() {
		x++
	}()
	return x // 返回值 = y = x =5
}
func f4() (x int) {
	defer func(x int) {
		x++ //改变的是函数中的副本
	}(x) //x当参数传递
	return 5 // 返回值 = x = 5
}

func main() {
	// deferDemo()
	fmt.Println(f1())
	fmt.Println(f2())
	// fmt.Println(f3())
	// fmt.Println(f4())
}
```

## 递归

```go
package main

import "fmt"

// 递归：函数自己调用自己

// 递归适合处理那种问题相同，规模越来越小的场景
// 递归一定要有一个明确的退出条件

func demo(n uint64) uint64 {
	if n <= 1 {
		return 1
	}
	return n * demo(n-1)
}

func main() {
	a := demo(5)
	fmt.Println(a)
}
```
