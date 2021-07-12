---
title: Go 基本结构和数据类型
date: 2020/12/10
description: Go 基本结构和数据类型
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/u9vgyrxMZbcAEG2.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/u9vgyrxMZbcAEG2.png'
categories:
  - go
tags:
  - go
  - 语法
abbrlink: 2317
---

# Go 基本结构和数据类型

**Go**（又称 **Golang**）是 Google 的 Robert Griesemer，Rob Pike 及 Ken Thompson 开发的一种**静态、强类型、编译型语言**。**Go** 语言语法与 **C** 相近，但功能上有：**内存安全**，**GC**（垃圾回收），**结构形态** 及 **CSP-style 并发计算**。

**Go**语言专门针对多处理器系统应用程序的编程进行了优化，使用Go编译的程序可以媲美C或C++代码的速度，而且更加安全、支持并行进程。

![image-20201210203134348](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/156oXytGlQVes8k.png)

## 一、基本架构

### 1. 名命

命名在所有的编程语言中都是有规则可寻的，也是需要遵守的，只有我们有了好的命名习惯才可以写出好的代码，例如我们在生活中对建筑的命名也是希望可以表达这个建筑的含义和作用。在Go语言中也是一样的，Go语言的函数名，变量名，常量名，类型名和包的命名也是都遵循这一规则的：一个一个名字必须以一个字母（Unicode字母）或下划线开头，后面可以跟任意数量的字母、数字或下划线。**大写字母和小写字母是不同的：Car和car是两个不同的名字。**

Go语言中也有类似java的关键字，且关键字不能用于自定义名字，只能在特定语法结构中使用。

~~~go
break      default        func     interface    select
case       defer          go       map          struct
chan       else           goto     package      switch
const      fallthrough    if       range        type
continue   for            import   return       var
~~~

除此之外Go语言中还有30多个预定义的名字，比如 **int** 和 **ture** 等

~~~go
内建常量: true false iota nil

内建类型: int   int8   int16   int32  int64   uint   uint8  uint16   uint32   uint64   uintptr  
         float32   float64   complex128   complex64   bool   byte    rune   string  error

内建函数: make  len  cap   new   append   copy   close   delete   complex   real    imag    panic  recover
~~~

> 通常我们在Go语言编程中推荐的命名方式是驼峰命名例如:ReadAll,不推荐下划线命名

### 2. 常量

在Go语言中，常量是指编译期间就已知且不可改变的值。常量可以是数值类型（包括整型、浮点型和复数类型）、布尔类型、字符串类型等。

下面我们使用const 关键字来定义常量:

~~~go
package main

import "fmt"
import "math"

// "const" 关键字用来定义常量
const s string = "appropriate"

func main() {    
    fmt.Println(s)
    
    // "const"关键字可以出现在任何"var"关键字出现的地方    
    // 区别是常量必须有初始值    
    const n = 20
    
    // 常量表达式可以执行任意精度数学计算    
    const d = 3e20 / n    
    fmt.Println(d)
    
    // 数值型常量没有具体类型，除非指定一个类型    
    // 比如显式类型转换    
    fmt.Println(int64(d))
    
    // 数值型常量可以在程序的逻辑上下文中获取类型    
    // 比如变量赋值或者函数调用。    
    // 例如，对于math包中的Sin函数,它需要一个float64类型的变量    
    fmt.Println(math.Sin(n))
}
~~~

输出的结果为：

~~~go
appropriate
6e+11
600000000000
-0.28470407323754404
~~~

### 3. 变量

通常用var声明语句可以创建一个特定类型的变量，然后给变量附加一个名字，并且设置变量的初始值。

Go的基本类型有：

~~~go
* bool
* string
* int int8 int16  int32 int64 
* uint uint8 uint16 uint32 uint64 uintptr
* byte // uint8 的别名
* rune // int32 的别名 代表一个Unicode码
* float32 float64
* complex64 complex128
~~~

变量的声明的语法一般是：

~~~go
var 变量名字 类型 = 表达式
~~~

通常情况下 **类型** 或 **= 表达式** 两个部分可以省略其中的一个。如果省略的是类型信息，那么将根据初始化表达式来推导变量的类型信息。如果初始化表达式被省略，那么将用零值初始化该变量。

- 数值类型变量对应的零值是0
- 布尔类型变量对应的零值是false
- 字符串类型对应的零值是空字符串
- 接口或引用类型（包括slice、map、chan和函数）变量对应的零值是nil
- 数组或结构体等聚合类型对应的零值是每个元素或字段都是对应该类型的零值。

零值初始化机制可以确保每个声明的变量总是有一个良好定义的值，因此在Go语言中不存在未初始化的变量。

通常我们在编程的过程中，也用简短声明变量。它以 **名字 := 表达式** 形式声明变量，变量的类型根据表达式来自动推导。

~~~go
destination := 12
result := rand.Float64() * 3.0
~~~

因为简洁和灵活的特点，简短变量声明被广泛用于大部分的局部变量的声明和初始化。
var形式的声明语句往往是用于需要显式指定变量类型地方，或者因为变量稍后会被重新赋值而初始值无关紧要的地方。

变量的生命周期指的是：**在程序运行期间变量有效存在的时间间隔**。

对于在包一级声明的变量来说，它们的生命周期和整个程序的运行周期是一致的。而相比之下，在局部变量的声明周期则是动态的：从每次创建一个新变量的声明语句开始，直到该变量不再被引用为止，然后变量的存储空间可能被回收。函数的参数变量和返回值变量都是局部变量。它们在函数每次被调用的时候创建。

### 4. 赋值

使用赋值语句可以更新一个变量的值

最简单的赋值语句是将要被赋值的变量放在 =的左边，新值的表达式放在 =的右边。

~~~go
x = 1 // 命令变量的赋值
*p = true // 通过指针间接赋值
person.name = "keke" // 结构体字段赋值
count[n] = count[n] * scale // 数组、slice或map的元素赋值
~~~

数值变量也可以支持++递增和--递减语句

~~~go
v := 1
v++ // 等价方式 v = v + 1；v 变成 2
v-- // 等价方式 v = v - 1；v 变成 1
~~~

### 5. 类型

变量或表达式的类型定义了对应存储值的属性特征，类型声明语句一般出现在包一级。

**因此如果新创建的类型名字的首字符大写，则在外部包也可以使用。**

~~~go
type 类型名字 底层类型

type Precision float64  #精确度
~~~

### 6. 包和文件

Go语言中的包和其他语言的库或模块的概念类似，目的都是为了支持**模块化、封装、单独编译和代码重用**。一个包的源代码保存在一个或多个以.go为文件后缀名的源文件中。在Go语言中包还可以让我们通过控制哪些名字是外部可见的来隐藏内部实现信息。在Go语言中，一个简单的规则是：

**如果一个名字是大写字母开头的，那么该名字是导出的。**

如果包中含有多个.go源文件，它们将按照发给编译器的顺序进行初始化，Go语言的构建工具首先会将 .go文件根据文件名排序，然后依次调用编译器编译。对于在包级别声明的变量，如果有初始化表达式则用表达式初始化，还有一些没有初始化表达式的，例如某些表格数据初始化并不是一个简单的赋值过程。

**在这种情况下，我们可以用一个特殊的init初始化函数来简化初始化工作。每个文件都可以包含多个 init 初始化函数**

~~~go
func init() { }
~~~

这样的 **init** 初始化函数除了不能被调用或引用外，其他行为和普通函数类似。在每个文件中的 **init** 初始化函数，在程序开始执行时按照它们声明的顺序被自动调用。
每个包在解决依赖的前提下，以导入声明的顺序初始化，每个包只会被初始化一次。

因此，如果一个p包导入了m包，那么在p包初始化的时候可以认为m包必然已经初始化过了。初始化工作是自下而上进行的，main包最后被初始化。以这种方式，可以确保在main函数执行之前，所有依然的包都已经完成初始化工作了。

import导入包的用法：

~~~go
import       "github.com/tidwall/gjson"   //通过包名gjson调用导出接口
import  json "github.com/tidwall/gjson"   //通过别名json调用gjson
import     . "github.com/tidwall/gjson"   //.符号表示，对包gjson的导出接口的调用直接省略包名
import    _  "github.com/tidwall/gjson"   //_ 仅仅会初始化gjson，如初始化全局变量，调用init函数
~~~

### 7. 作用域

一个声明语句将程序中的实体和一个名字关联，比如一个函数或一个变量。声明语句的作用域是指源代码中可以有效使用这个名字的范围。不要将作用域和生命周期混为一谈。声明语句的作用域对应的是一个源代码的文本区域；它是一个编译时的属性。一个变量的生命周期是指程序运行时变量存在的有效时间段，在此时间区域内它可以被程序的其他部分引用；是一个运行时的概念。

语法块是由花括弧所包含的一系列语句，就像函数体或循环体花括弧对应的语法块那样。语法块内部声明的名字是无法被外部语法块访问的。语法决定了内部声明的名字的作用域范围。我们可以这样理解，语法块可以包含其他类似组批量声明等没有用花括弧包含的代码，我们称之为语法块。有一个语法块为整个源代码，称为全局语法块；然后是每个包的包语法决；每个for、if和switch语句的语法决；每个
switch或select的分支也有独立的语法决；当然也包括显式书写的语法块（花括弧包含的语句）。

声明语句对应的词法域决定了作用域范围的大小。对于内置的类型、函数和常量，比如int、len和true等是在全局作用域的，因此可以在整个程序中直接使用。任何在在函数外部（也就是包级语法域）声明的名字可以在同一个包的任何源文件中访问的。对于导入的包，例如tempconv导入的fmt包，则是对应源文件级的作用域，因此只能在当前的文件中访问导入的fmt包，当前包的其它源文件无法访问在当前源文件导入的包

在包级别，声明的顺序并不会影响作用域范围，因此一个先声明的可以引用它自身或者是引用后面的一个声明，这可以让我们定义一些相互嵌套或递归的类型或函数。但是如果一个变量或常量递归引用了自身，则会产生编译错误。

## 二、基本数据类型

### 1. 整型

Go语言同时提供了有符号和无符号类型的整数运算。

~~~go
有符号整形数类型:
int8,长度:1字节, 取值范围:(-128 ~ 127)
int16,长度:2字节,取值范围:(-32768 ~ 32767）
int32,长度:4字节,取值范围:(-2,147,483,648 ~ 2,147,483,647）
int64.长度:8字节,取值范围:(-9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807)

无符号整形数类型:
uint8,长度:1字节, 取值范围:(0 ~ 255)
uint16,长度:2字节,取值范围:(0 ~ 65535)
uint32,长度:4字节,取值范围:(0 ~ 4,294,967,295)
uint64.长度:8字节,取值范围:(0 ~ 18,446,744,073,709,551,615)
~~~

字符是 UTF-8 编码的 Unicode 字符，Unicode 为每一个字符而非字形定义唯一的码值（即一个整数），例如 字符a 在 unicode 字符表是第 97 个字符，所以其对应的数值就是 97，也就是说对于Go语言处理字符时，97 和 a 都是指的是字符a，而 Go 语言将使用数值指代字符时，将这样的数值称呼为 rune 类型。
rune类型是 Unicode 字符类型，和 int32 类型等价，通常用于表示一个 Unicode 码点。rune 和 int32 可以互换使用。
一个Unicode代码点通常由"U+"和一个以十六进制表示法表示的整数表示，例如英文字母'A'的Unicode代码点为"U+0041"。

此外rune类型的值需要由单引号"'"包裹，不过我们还可以用另外几种方式表示: 

| 示例                    | 备注                    | 关键字                                 |
| ----------------------- | ----------------------- | -------------------------------------- |
| 直接使用**Unicode**字符 | "郝" 表示 中文 "郝"     | 只能表示**Unicode**支持的字符          |
| "\x" 加两个十六进制数   | "\x41" 表示英文字母 "A" | 只能表示**ASCII**编码表示的字符        |
| "\\" 加三位八进制数     | "\101" 表示英文字母 "A" | 只能表示编码值在 **[0, 255)** 的字符   |
| "\u" 加四位十六进制数   | "\u90DD" 表示 "郝"      | 只能表示编码值在 **[0，65535)** 的字符 |
| "\U" 加四位十六进制数   | "\U000090DD" 表示 "郝"  | 可以用于表示任何 **Unicode** 字符      |

byte是uint8类型的等价类型，byte类型一般用于强调数值是一个原始的数据而不是 一个小的整数。

uintptr 是一种无符号的整数类型，没有指定具体的bit大小但是足以容纳指针。 uintptr类型只有在底层编程是才需要，特别是Go语言和C语言函数库或操作系统接口相交互的地方。

此外在这里还需要了解下进制的转换方便以后学习和使用：

~~~go
十进制整数: 使用0-9的数字表示且不以0开头。// 100 123455
八进制整数: 以0开头，0-7的数字表示。 // 0100 0600
十六进制整数: 以0X或者是0x开头，0-9|A-F|a-f组成 //0xff 0xFF12
~~~

### 2. 浮点型

浮点型。float32 精确到小数点后 7 位，float64 精确到小数点后 15 位。由于精确度的缘故，你在使用 == 或者 != 来比较浮点数时应当非常小心。

```go
浮点型（IEEE-754 标准）:
float32:（+- 1e-45 -> +- 3.4 * 1e38）32位浮点类型

float64:（+- 5 1e-324 -> 107 1e308）64位浮点类型
```

浮点型中指数部分由"E"或"e"以及带正负号的10进制整数表示。例:3.9E-2表示浮点数0.039。3.9E+1表示浮点数39。有时候浮点数类型值也可以被简化。比如39.0可以被简化为39。0.039可以被简化为.039。在Golang中浮点数的相关部分只能由10进制表示法表示。

### 3. 复数

~~~go
复数类型:
complex64: 由两个float32类型的值分别表示复数的实数部分和虚数部分

complex128: 由两个float64类型的值表示复数的实数部分和虚数部分
~~~

复数类型的值一般由浮点数表示的实数部分、加号"+"、浮点数表示的虚数部分以及小写字母"i"组成，例如：

~~~go
 var x complex128 = complex(1,2)  //1+2i
~~~

对于一个复数 c =  complex(x, y) ，可以通过Go语言内置函数 real(z) 获得该复数的实
部，也就是 x ，通过 imag(c) 获得该复数的虚部，也就是 y 。

### 4. 布尔型

在Go语言中，布尔值的类型为 bool，值是 true 或 false,布尔可以做3种逻辑运算，&&（逻辑且），||（逻辑或），！（逻辑非）,布尔类型的值不支持其他类型的转换.

布尔值可以和&&(AND)和||(OR)操作符结合,并且可能会有短路行为:如果运算符左边值已经可以确定整个布尔表达式的值,那么运算符右边的值将不在被求值,因此下面的表达式总是安全的:

~~~go
 s != "" && s[0] == 'x'
~~~

其中 s[0] 操作如果应用于空字符串将会导致panic异常。

### 5. 字符串

在Go语言中，组成字符串的最小单位是字符，存储的最小单位是字节，**字符串本身不支持修改**。字节是数据存储的最小单元，每个字节的数据都可以用整数表示，例如一个字节储存的字符a，实际存储的是97而非字符的字形，将这个实际存储的内容用数字表示的类型，称之为byte。

**字符串是不可变的字节序列**，它可以包含任意数据，包括0值字节，但是主要还是为了人可读的文本。内置的 len()函数返回字符串的字节数。

 字符串的表示法有两种，即：原生表示法和解释型表示法。原生表示法，需用用反引号"`"把字符序列包起来，如果用解释型表示法，则需要用双引号"""包裹字符序列。

~~~go
var str1 string = "keke"
var str2 string = `keke`
~~~

这两种表示的区别是，前者表示的是所见即所得的(除了回车符)。后者所表示的值中转义符会起作用。字符串值是不可变的，如果我们创建了一个此类型的值，就不可能再对它本身做任何修改。

~~~go
var str string  // 声明一个字符串变量
str = "hai keke" // 字符串赋值
ch := str[0] // 取字符串的第一个字符
~~~

### 6. 常量

常量表达式的值在编译期计算,而不是在运行期。每种常量的潜在类型都是基础类型:boolean、string或数字。

~~~go
const x, y int = 1, 2 // 多常量初始化
const s = "Hello, KeKe!" // 类型推断
const ( // 常量组
a, b
c
= 10, 20
bool = false
)

func main() {

const m = "20"// 未使用用局部常量不会引发编译错误。
}
~~~

**枚举**：关键字 iota 定义常量组中从 0 开始按行行计数的自自增枚举值。

iota在const关键字出现时将被重置为0(const内部的第一行之前)，const中每新增一行常量声明将使iota计数一次(iota可理解为const语句块中的行索引)。使用iota能简化定义，在定义枚举时很有用。

~~~go
const (
 Sunday = iota // 0
 Monday // 1,通常省略后续行行表达式。
 Tuesday // 2
 Wednesday // 3
 Thursday // 4
 Friday // 5
 Saturday // 6
)
~~~

在同一常量组中,可以提供多个 iota,它们各自增⻓。

~~~go
const (
 A, B = iota, iota << 10 // 0, 0 << 10
 C, D // 1, 1 << 10
)
~~~

容量大小的单位的自增：

~~~go
const (
     B = 1 << (10*iota)
     KB
     MB
     GB
     TB
     PB
)
~~~

bit就是位，也叫比特位，是计算机表示数据最小的单位,byte是字节。

- 1B（byte，字节,1byte就是1B）= 8 bit(位就是bit也是b)
- 一个字符=2字节(2byte)

### 7. 整型运算

在整型运算中，算术运算、逻辑运算和比较运算，运算符优先级从上到下递减顺序排列：

~~~go
 *      /     %     <<     >>     &     &^ 
 +      -     |     ^      
 ==     !=    <     <=     >      >=
 &&     
 ||
~~~

在同一个优先级，使用左优先结合规则，但是使用括号可以明确优先顺序。

bit位操作运算符：

| 符号 | 操作             | 操作数是否区分符号 |
| ---- | ---------------- | ------------------ |
| &    | 位运算 AND       | No                 |
| ^    | 位运算 XOR       | No                 |
| &^   | 位清空 (AND NOT) | No                 |
| <<   | 左移             | Yes                |
| >>   | 右移             | Yes                |

## 三、复合数据类型

### 1. 数组

在复合数据类型中数组是由同构的元素组成——每个数组元素都是完全相同的类型——结构体则是由异构的元素组成的。数组和结构体都是有固定内存大小的数据结构。相比之下，slice和map则是动态的数据结构，它们将根据需要动态增长。

数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。由于数组的长度是固定的，因而在使用的时候我们用的最多的是slice（切片），它是可以增长和收缩动态序列，slice功能也更灵活，但是要理解slice工作原理的话需要先理解数组。

数组的每个元素可以通过索引下标来访问，索引下标的范围是从0开始到数组长度减1的位置。内置的len函数将返回数组中元素的个数。

数组的每个元素都被初始化为元素类型对应的零值，对于数字类型来说就是0。

~~~go
var m [3]int = [3]int{1, 2, 3}
var n [3]int = [3]int{1, 2}
fmt.Println(n[2]) // "0"
~~~

在数组字面值中，如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算。

~~~go
m := [...]int{1, 2, 3}
fmt.Printf("%T\n", m) // "[3]int"
~~~

数组的长度是数组类型的一个组成部分，因此[3]int和[4]int是两种不同的数组类型。数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定。

数组可以直接进行比较，当数组内的元素都一样的时候表示两个数组相等。

~~~go
arr1 := [3]int{1, 2, 3}
arr2 := [3]int{1, 2, 3}
arr3 := [3]int{1, 2, 4} 
fmt.Println(arr1 == arr2, arr1 == arr3)  //true,false
~~~

数组可以作为函数的参数传入，但由于数组在作为参数的时候，其实是进行了拷贝，这样在函数内部改变数组的值，是不影响到外面的数组的值得。

~~~go
func ArrIsArgs(arr [4]int) {
    arr[0] = 120
}
m := [...]int{1, 2, 3, 4}
ArrIsArgs(m)
~~~

如果想要改变就只能使用指针，在函数内部改变的数组的值，也会改变外面的数组的值：

~~~go
func ArrIsArgs(arr *[4]int) {
   arr[0] = 20
}
m:= [...]int{1, 2, 3, 4}
ArrIsArgs(&m)
~~~

通常这样的情况下都是用切片来解决，而不是用数组。

这里的`*` 和`&`的区别:

* `&` 是取地址符号 , 即取得某个变量的地址 ,如: &a
* `*` 是指针运算符 , 可以表示一个变量是指针类型 , 也可以表示一个指针变量所指向的存储单元 , 也就是这个地址所存储的值。

### 2. Slice

#### slice 的切片操作

代表变长的序列，序列中每个元素都有相同的类型。一个slice类型一般写作`[]T`,其中T代表slice中元素的类型；
slice的语法和数组很像，只是没有固定长度而已。

数组和slice关系非常密切，一个slice可以访问数组的部分或者全部数据，而且slice的底层本身就是对数组的引用。

一个Slice由三部分组成：指针，长度和容量。内置的 **len** 和 **cap** 函数可以分别返回 slice 的长度和容量。
指针指向第一个slice元素对应的底层数组元素的地址，**要注意的是slice的第一个元素并不一定就是数组的第一个元素**。
长度对应 slice 中元素的数目；长度不能超过容量，容量一般是从slice的开始位置到底层数据的结尾位置。

多个 slice 之间可以共享底层的数据,并且引用的数组部分区间可能重叠。

slice 的切片操作 `s[i:j]`，其中 **0 ≤ i≤ j≤ cap(s)** 用于创建一个新的 slice，引用s的从第 i 个元素开始到第 j-1 个元素的子序列。
新的 slice 将只有 j-i 个元素。如果 i 位置的索引被省略的话将使用0代替，如果j位置的索引被省略的话将使用 len(s) 代替。
**如果切片操作超出 cap(s) 的上限将导致一个 panic 异常，但是超出 len(s) 则是意味着扩展了 slice，因为新 slice 的长度会变大。**

~~~go
func main(){
   p:=[]int{1,2,3}
   fmt.Println("p:",p)

   m1 :=p[:2]
   fmt.Println("m1:",m1)

   m2 := m1[1:]
   fmt.Println("m2:",m2)

   m3 := m2[:2]
   m4 := p[:2][1:][:2]
   fmt.Println("m3",m3)
   fmt.Println("m4",m4)
}

结果如下:

p: [1 2 3]
m1: [1 2]
m2: [2]
m3 [2 3]
m4 [2 3]
~~~

slice 的底层本身就是对数组的引用，因而多个 slice 共享的是底层的同一个数组。如果你**指定了截止的位置并且这个位置没有超过底层数组的范围，它就会指针引用底层的数组**，这还是可以取到底层数组的值。

#### slice 创建方式

- 直接创建：内建函数new分配了零值填充的元素类型的内存空间，并且返回其地址，一个指针类型的值。

  ~~~go
  var p *[]int = new([]int)  //分配slice结构内存
  var ｍ []int = make([]int,100) //m指向一个新分配的有100个整数的数组
  
  // new 分配; make 初始化
  new(T) 返回 *T 指向一个零值 T
  make(T) 返回初始化后的 T
  ~~~

- 内置的make函数创建一个指定元素类型、长度和容量的slice。容量部分可以省略,在这种情况下,容量将等于长度。

  ~~~go
  // 使用内置的make()函数来创建。事实上还是会创建一个匿名的数组，只是不需要我们来定义。
  make([]T, len)
  make([]T, len, cap) // 类似 make([]T, cap)[:len]
  
  // 在底层,make创建了一个匿名的数组变量,然后返回一个slice;只有通过返回的slice才能引用底层匿名的数组变量。在第一种语句中,slice是整个数组的view。在第二个语句中,slice只引用了底层数组的前len个元素,但是容量将包含整个的数组。额外的元素是留给未来的增长用的。
  
  slice1 := make([]int,5)//创建一个元素个数5的slice,cap也是5
  slice2 := make([]int,5,10)//创建一个元素个数5的slice，cap是10
  slice3 := []int{1,2,3,4,5}//创建一个元素个数为5的slice，cap是5
  var slice []int //创建一个空的slice，cap和len都是0
  ~~~

#### slice 的比较

**slice之间不能比较**，因此我们不能使用 == 操作符来判断两个slice是否含有全部相等元素。
不过标准库提供了高度优化的bytes.Equal函数来判断两个**字节型slice是否相等([]byte)**，但是对于其他类型的slice,我们必须自己展开每个元素进行比较

~~~go
func equal(x, y []string) bool {   
    if len(x) != len(y) {        
        return false   
    }   
    for i := range x {        
        if x[i] != y[i] {            
            return false        
        }  
    }  
    return true
}
~~~

> 通过在slice的深度相等测试,运行的时间并不比支持==操作的数组或字符串更多,但是为何slice不直接支持比较运算符呢?这方面有两个原因。第一个原因,一个slice的元素是间接引用的,一个slice甚至可以包含自身。虽然有很多办法处理这种情形,但是没有一个是简单有效的。
>
> 第二个原因,因为slice的元素是间接引用的,一个固定值的slice在不同的时间可能包含不同的元素,因为底层数组的元素可能会被修改。并且Go语言中map等哈希表之类的数据结构的key只做简单的浅拷贝,它要求在整个声明周期中相等的key必须对相同的元素。对于像指针或chan之类的引用类型,==相等测试可以判断两个是否是引用相同的对象。一个针对slice的浅相等测试的==操作符可能是有一定用处的,也能临时解决map类型的key问题,但是slice和数组不同的相等测试行为会让人困惑。因此,安全的做法是直接禁止slice之间的比较操作。

slice唯一合法的比较操作是和nil比较,例如:

```go
if slice == nil { /* ... */ }
```

一个零值的slice 等于 nil。一个 nil 值的slice并没有底层数组。一个nil值的slice的长度和容量都是0
但是也有非nil值的slice的长度和容量也是0的，例如 []int{} 或 make([]int, 3)[3:]。与任意类型的nil值一样,我们可以用 []int (nil) 类型转换表达式来生成一个对应类型 slice 的nil值。

```go
var s []int      // len(s) == 0, s == nil
s = nil         // len(s) == 0, s == nil
s = []int(nil)  // len(s) == 0, s == nil
s = []int{}     // len(s) == 0, s != nil
```

如果你需要测试一个 slice 是否是空的,使用len(s) == 0来判断,而不应该用s == nil 来判断。
除了和 nil 相等比较外。一个 nil 值的 slice 的行为和其它任意0长度的slice一样.

#### slice 追加元素

Slice 切片有 append 函数用于向 slice 追加元素。
虽然Slice是可以动态扩展的，但Slice的动态扩展是有代价的，也就是说如果在确定大小的前提下，最好是设置好 slice 的 cap 大小.

~~~go
func main() {
    var x, y []int
    for i := 0; i < 10; i++ {
        y = appendInt(x, i)
        fmt.Printf("%d cap=%d\t%v\n", i, cap(y), y)
        x = y
    }
}

// 每一次容量的变化都会导致重新分配内存和copy操作
0 cap=1 [0]
1 cap=2 [0 1]
2 cap=4 [0 1 2]
3 cap=4 [0 1 2 3]
4 cap=8 [0 1 2 3 4]
5 cap=8 [0 1 2 3 4 5]
6 cap=8 [0 1 2 3 4 5 6]
7 cap=8 [0 1 2 3 4 5 6 7]
8 cap=16 [0 1 2 3 4 5 6 7 8]
9 cap=16 [0 1 2 3 4 5 6 7 8 9]
~~~

append的底层原理就是当slice的容量满了的时候，重新建立一块内存，然后将原来的数据拷贝到新建的内存。所以说容量的扩充是存在内存的建立和复制的。该过程将会影响到系统的运行速度。

更新slice变量不仅对调用append函数是必要的,实际上对应任何可能导致长度、容量或底层数组变化的操作都是必要的。要正确地使用slice，需要记住尽管底层数组的元素是间接访问的,但是slice对应结构体本身的指针、长度和容量部分是直接访问的。要更新这些信息需要像上面例子那样一个显式的赋值操作。从这个角度看,slice并不是一个纯粹的引用类型,它实际上是一个类似下面结构体的聚合类型:：

~~~go
type IntSlice struct {
    ptr         *int
    len, cap    int
}
~~~

我们的 appendInt 函数每次只能向slice追加一个元素，但是内置的append函数则可以追加多个元素,甚至追加一个slice。

~~~go
var x []int
x = append(x, 1)
x = append(x, 2, 3)
x = append(x, 4, 5, 6)
x = append(x, x...) // append the slice x
fmt.Println(x)     // "[1 2 3 4 5 6 1 2 3 4 5 6]"
~~~

内置的copy函数可以方便地将一个slice复制另一个相同类型的slice。

~~~go
slice1 := []int{1, 2, 3, 4, 5}
slice2 := []int{5, 4, 3}
copy(slice2, slice1) // 只会复制slice1的前3个元素到slice2中
copy(slice1, slice2) // 只会复制slice2的3个元素到slice1的前3个位置
~~~

copy函数的第一个参数是要复制的目标slice，第二个参数是源slice，目标和源的位置顺序和dst = src赋值语句是一致的。
两个slice可以共享同一个底层数组,甚至有重叠也没有问题。copy函数将返回成功复制的元素的个数(我们这里没有用到)，等于两个slice中较小的长度,所以我们不用担心覆盖会超出目标slice的范围。

#### slice使用技巧

一个slice可以用来模拟一个stack。

~~~go
// 最初给定的空slice对应一个空的stack,然后可以使用append函数将新的值压入stack
stack = append(stack, v)
// stack的顶部位置对应slice的最后一个元素
top := stack[len(stack)-1]
// 通过收缩stack可以弹出栈顶的元素
stack = stack[:len(stack)-1]
~~~

要删除slice中间的某个元素并保存原有的元素顺序,可以通过内置的copy函数将后面的子slice向前依次移动一位完成

~~~go
func remove(slice []int, i int) []int {
    copy(slice[i:], slice[i+1:])
    return slice[:len(slice)-1]
}
func main() {
    s := []int{5, 6, 7, 8, 9}
    fmt.Println(remove(s, 2)) // "[5 6 8 9]"
}
~~~

如果删除元素后不用保持原来顺序的话,我们可以简单的用最后一个元素覆盖被删除的元素

~~~go
func remove(slice []int, i int) []int {
    slice[i] = slice[len(slice)-1]
    return slice[:len(slice)-1]
}
func main() {
    s := []int{5, 6, 7, 8, 9}
    fmt.Println(remove(s, 2)) // "[5 6 9 8]
}
~~~

如果要反转整个slice的元素

~~~go
s := []int{0, 1, 2, 3, 4, 5}
reverse(s)
fmt.Println(s) // "[5 4 3 2 1 0]"
~~~

slice 有个特性是允许多个 slice 指向同一个底层数组，这是一个有用的特性，在很多场景下都能通过这个特性实现 no copy而提高效率。**但共享同时意味着不安全。因为slice是有长度和容量的，在没有扩容的情况下,可能改变了结构**

为了避免这样的情况的发生我们通常是建议使用make去指定len和cap
或者使用到slice的截取的特性：`slice[0:1:1]` ([起始index，终止index，cap终止index]).

### 3. Map

映射Map是一个存储键值对的无序集合，映射Map是一种巧妙并且实用的数据结构。它是一个无序的key/value对的集合,其中所有的key都是不同的，然后通过给定的key可以在常数时间复杂度内检索、更新或删除对应的value。

在Go语言中，一个map就是一个映射Map的引用，map类型可以写为`map[K]V`,其中K和V分别对应key和value。map中所有的key都有相同的类型。所有的value也有着相同的类型，但是key和value之间可以是不同的数据类型。**其中K对应的key必须是支持  == 比较运算符的数据类型**，所以map可以通过测试key是否相等来判断是否已经存在。

#### Map的声明

~~~go
var m map[string] string
// m是声明的变量名，string是对应的Key的类型，string是value的类型,因此这个是声明了一个key和value都是string的map
~~~

#### 创建

~~~go
// Go内置的make函数可以创建map
m := make(map[string]int)
m["keke"] = 001
m["jame"] = 002
// 我们也可以用map字面值的语法创建map,同时还可以指定一些最初的key/value:
m := map[string]int{
"keke": 001,
"jame": 002,
}
~~~

#### 元素的删除

~~~go
// Map可以使用内置的delete函数可以删除元素
delete(m, "jame") 

// 而且x += y和x++等简短赋值语法也可以用在map上
m["keke"] += 1
m["keke"]++

// 但是map中的元素并不是一个变量,因此我们不能对map的元素进行取址操作
_ = &m["keke"] // compile error: cannot take address of map element
禁止对map元素取址的原因是map可能随着元素数量的增长而重新分配更大的内存空间,从而可能导致之前的地址无效
~~~

#### 元素取址

**map中的元素并不是一个变量，因此我们不能对map的元素进行取址操作**
禁止对map元素取址的原因是map可能随着元素数量的增长而重新分配更大的内存空间，从而可能导致之前的地址无效

~~~go
_ = &m["keke"] // compile error: cannot take address of map element
~~~

#### 遍历 map

要想遍历map中全部的key/value对的话，可以使用range风格的for循环实现，和之前的slice遍历语法类似。下面的迭代语句将在每次迭代时设置name和age变量，它们对应下一个键/值对:

~~~go
for k, v := range m {
    fmt.Printf("%s\t%d\n", k, v)
}
~~~

Map的迭代顺序是不确定的，并且不同的哈希函数实现可能导致不同的遍历顺序。**在实践中，遍历的顺序是随机的，每一次遍历的顺序都不相同。**这是故意的，每次都使用随机的遍历顺序可以强制要求程序不会依赖具体的哈希函数实现。如果要按顺序遍历key/value对,我们必须显式地对key进行排序,可以使用sort包的Strings函数对字符串slice进行排序。常见的处理方式:

~~~go
import "sort"

var names []string
for name := range ages {
    names = append(names, name)
}
sort.Strings(names)
for _, name := range names {
    fmt.Printf("%s\t%d\n", name, ages[name])
}
~~~

#### 零值

map类型的零值是 nil，也就是没有引用任何映射Map。

~~~go
var ages map[string]int
fmt.Println(ages == nil)  // "true"
fmt.Println(len(ages) == 0) // "true"
~~~

### 4. 结构体

#### 结构体声明

Go提供的结构体就是把使用各种数据类型定义的不同变量组合起来的高级数据类型。

~~~go
type Rectangle struct {
     width float64
     length float64
}
// 通过type定义一个新的数据类型，然后是新的数据类型名称rectangle，最后是struct关键字，表示这个高级数据类型是结构体类型。
~~~

其实构体类型和基础数据类型使用方式差不多，唯一的区别就是结构体类型可以通过`.`来访问内部的成员。包括给内部成员赋值和读取内部成员值。

~~~go
package main

import (
	"fmt"
)

type Rect struct {
	width  float64
    	length float64
}

func main() {
    var a Rectangle
    a.width = 100
    a.length = 200
	
    // 使用初始化的方式来给rectangle变量的内部成员赋值。
    var b = Rectangle{width: 100, length: 200}
    
    // 知道结构体成员定义的顺序，也可以不使用key:value的方式赋值，直接按照结构体成员定义的顺序给它们赋值
    var c = Rectangle{100, 200}
}
~~~

- 如果结构体成员名字是以大写字母开头的，那么该成员就是导出的；这是Go语言导出规则决定的。一个结构体可能同时包含导出和未导出的成员。直白的讲就是首字母大写的结构体字段可以被导出，也就是说，在其他包中可以进行读写。结构体字段名以小写字母开头是当前包的私有的，函数定义也是类似的。

- 如果结构体没有任何成员的话就是空结构体，写作`struct{}`。它的大小为0，也不包含任何信息，但是有时候依然是有价值的。有些Go开发者用map模拟set数据结构时，用它来代替map中布尔类型的value，只是强调key的重要性，但是因为节约的空间有限，而且语法比较复杂,所有我们通常避免避免这样的用法。

- 一个命名为S的结构体类型将不能再包含S类型的成员：因为一个聚合的值不能包含它自身（该限制同样适应于数组）。**但是S类型的结构体可以包含S指针类型的成员，这可以让我们创建递归的数据结构，比如链表和树结构等。**

  ~~~go
  type tree struct {
      value       int
      left, right *tree
  }
  ~~~

#### 结构体嵌入和匿名成员

~~~go
type Point struct {
    X, Y int
}

type Circle struct {
    Center Point
    Radius int
}

type Wheel struct {
    Circle Circle
    Spokes int
}

// 访问每个成员
var w Wheel
w.Circle.Center.X = 8
w.Circle.Center.Y = 8
w.Circle.Radius = 5
w.Spokes = 20
~~~

~~~go
type Point struct {
    X, Y int
}

type Circle struct {
    Point  // 匿名字段，struct
    Radius int
}

type Wheel struct {
    Circle  // 匿名字段，struct
    Spokes int
}

var w Wheel
w.X = 8            // w.Circle.Point.X = 8
w.Y = 8            // w.Circle.Point.Y = 8
w.Radius = 5       // w.Circle.Radius = 5
w.Spokes = 20

// 结构体字面值并没有简短表示匿名成员的语法, 因此下面的语句都不能编译通过
w = Wheel{8, 8, 5, 20}   // compile error: unknown fields
w = Wheel{X: 8, Y: 8, Radius: 5, Spokes: 20} // compile error: unknown fields
~~~

**struct 不仅仅能够将 struct 作为匿名字段，自定义类型、内置类型都可以作为匿名字段，而且可以在相应的字段上面进行函数操作。**

#### 结构体tag

在Golang中结构体和数据库表的映射关系的建立是通过 struct Tag 来实现的。

~~~go
package main
import (
      "fmt"
      "reflect" // 这里引入reflect模块
)

type User struct {
     Name    string `json:"name"`  //这引号里面的就是tag
     Passwd  int    `json:"passwd"`
}
func main() {
     user := &User{"keke", 123456}
     s := reflect.TypeOf(user).Elem() //通过反射获取type定义
     for i := 0; i < s.NumField(); i++ {
         fmt.Println(s.Field(i).Tag.Get("json")) //将tag输出出来
     }
 }
~~~

结构体的成员Tag可以是任意的字符串面值，但是通常是一系列用空格分隔的key:"value"键值对序列；因为值中含义双引号字符，因此成员Tag一般用原生字符串面值的形式书写。

这里有必要解释下指针这块,在Golang中很多时候都是需要用指针结合结构体开发的。 通常指针是存储一个变量的内存地址的变量。

在Golang中，指针不参与计算但是可以用来获取地址的，例如变量a的内存地址为&a,这里的&就是获取a的地址。如果一个指针，它的值是在别的地方的地址，而且我们想要获取这个地址的值，可以使用*符号。*符号是为取值符。例如上面的&a是一个地址，那么这个地址里存储的值为*&a。


注意:这里的 & 和 * 的区别,& 运算符,用来获取指针地址,而 * 运算符是用来获取地址里存储的值。此外指针的值和指针的地址是不同的概念,指针的值: 指的是一个地址，是别的内存地址。指针的地址: 指的是存储指针内存块的地址。

通常 & 运算符是对变量取地址，如：变量a的地址是&a, * 运算符对指针取值,如:`*&a`，就是a变量所在地址的值，也就是a的值.此外 ＆和 * 以互相抵消,同时注意，` *& ` 可以抵消掉，但` &* `是不可以抵消的, a和` *&a`是一样的，都是a的值，因为` *& `互相抵消掉了，同理a和` *&*&*&*&a `是一样的 (因为4个` *& `互相抵消掉了)。

通常我们传一个参数值到被调用函数里面时，实际上是传了这个值的一份copy，当在被调用函数中修改参数值的时候，调用函数中相应实参不会发生任何变化，因为数值变化只作用在copy上。

传指针比较轻量级 (*bytes)，只是传内存地址，我们可以用指针传递体积大的结构体。如果用参数值传递的话， 在每次copy上面就会花费相对较多的系统开销（内存和时间）。**所以当你要传递大的结构体的时候，用指针是一个明智的选择。**

**Golang中string，slice，map这三种类型的实现机制类似指针，所以可以直接传递，而不用取地址后传递指针。**

注意：若函数需改变slice的长度，则仍需要取地址传递指针。

如果要访问指针 p 指向的结构体中某个元素 x，不需要显式地使用 * 运算，可以直接 p.x；

### 5. JSON

JavaScript对象表示法(JSON)是一种用于发送和接收结构化信息的标准协议。在类似的协议中，JSON并不是唯一的一个标准协议。 XML(§7.14)、ASN.1和Google的Protocol Buffers都是类似的协议，并且有各自的特色，但是由于简洁性、可读性和流行程度等原因，JSON是应用最广泛的一个。

JSON是对JavaScript中各种类型的值——字符串、数字、布尔值和对象——Unicode本文编码。

基本的JSON类型有数字(十进制或科学记数法)、布尔值(true或false)、字符串，其中字符串是以双引号包含的Unicode字符序列，支持和Go语言类似的反斜杠转义特性，不过JSON使用的是\Uhhhh转义数字来表示一个UTF-16编码(译注:UTF-16和UTF-8一样是一种变长的编码，有些Unicode码点较大的字符需要用4个字节表示；而且UTF-16还有大端和小端的问题)，而不是Go语言的rune类型。

这些基础类型可以通过JSON的数组和对象类型进行递归组合。一个JSON数组是一个有序的值序列，写在一个方括号中并以逗号分隔；一个JSON数组可以用于编码Go语言的数组和slice。一个JSON对象是一个字符串到值的映射,写成以系列的name:value对形式，用花括号包含并以逗号分隔；JSON的对象类型可以用于编码Go语言的map类型(key类型是字符串)和结构体。

#### 解析JSON

- 数据结构 --> 指定格式 = 序列化 或 编码（传输之前）
- 指定格式 --> 数据格式 = 反序列化 或 解码（传输之后）

序列化是在内存中把数据转换成指定格式（data -> string），反之亦然（string -> data structure）。
编码也是一样的，只是输出一个数据流（实现了 io.Writer 接口）；解码是从一个数据流（实现了 io.Reader）输出到一个数据结构。

在Go中我们经常需要做数据结构的转换，Go中提供的处理json的标准包是 encoding/json,主要使用的是以下两个方法：

~~~go
// 序列化 结构体=> json  
func Marshal(v interface{}) ([]byte, error)

// 反序列化 json=>结构体
func Unmarshal(data []byte, v interface{}) error

序列化前后的数据结构有以下的对应关系：
bool    for JSON  booleans
float64 for JSON  numbers
string  for JSON strings
[]interface{}  for JSON arrays
map[string]interface{}  for JSON objects
nil     for JSON null
~~~

#### json解析到结构体

如果我们有一段json要转换成结构体就需要用到(Unmarshal)样的函数：

~~~go
func Unmarshal(data []byte, v interface{}) error
~~~

~~~go
package main

import (
	"encoding/json"
	"fmt"
)

type Server struct {
      ServerName string
      ServerIP   string
}

type Serverslice struct {
      Servers []Server
}

func main() {
     var s Serverslice
     str := `{"servers":[{"serverName":"Shanghai","serverIP":"127.0.0.1"},	{"serverName":"Beijing","serverIP":"127.0.0.2"}]}`
     
     json.Unmarshal([]byte(str), &s)
     fmt.Println(s)
}
~~~

我们首先定义了与json数据对应的结构体，数组对应slice，字段名对应JSON里面的KEY，在解析的时候，如何将json数据与struct字段相匹配呢？例如JSON的key是Foo，那么怎么找对应的字段呢？这就用到了我们上面结构体说的tag。

**同时能够被赋值的字段必须是可导出字段(即首字母大写）**。同时JSON解析的时候只会解析能找得到的字段，找不到的字段会被忽略，这样的一个好处是：当你接收到一个很大的JSON数据结构而你却只想获取其中的部分数据的时候，你只需将你想要的数据对应的字段名大写，即可轻松解决这个问题。

#### 结构体转json

结构体转json就需要用到JSON包里面通过Marshal函数来处理：

~~~go
func Marshal(v interface{}) ([]byte, error)
~~~

~~~go
package main

import (
	"encoding/json"
	"fmt"
)

type Server struct {
	ServerName string
	ServerIP   string
}

type Serverslice struct {
	Servers []Server
}

func main() {
	var s Serverslice
	s.Servers = append(s.Servers, Server{ServerName: "Shanghai", ServerIP: "127.0.0.1"})
	s.Servers = append(s.Servers, Server{ServerName: "Beijing", ServerIP: "127.0.0.2"})
	b, err := json.Marshal(s)
	if err != nil {
		fmt.Println("json err:", err)
	}
	fmt.Println(string(b))
}

// {"Servers":[{"ServerName":"Shanghai","ServerIP":"127.0.0.1"},{"ServerName":"Beijing","ServerIP":"127.0.0.2"}]}
~~~

我们看到上面的输出字段名的首字母都是大写的，如果你想用小写的首字母怎么办呢？把结构体的字段名改成首字母小写的？JSON输出的时候必须注意，只有导出的字段才会被输出，如果修改字段名，那么就会发现什么都不会输出，所以必须通过struct tag定义来实现：

~~~go
type Server struct {
	ServerName string `json:"serverName"`
	ServerIP   string `json:"serverIP"`
}

type Serverslice struct {
	Servers []Server `json:"servers"`
}
~~~

针对JSON的输出，我们在定义struct tag的时候需要注意的几点是:

- 字段的tag是"-"，那么这个字段不会输出到JSON
- tag中带有自定义名称，那么这个自定义名称会出现在JSON的字段名中，例如上面例子中serverName
- tag中如果带有"omitempty"选项，那么如果该字段值为空，就不会输出到JSON串中
- 如果字段类型是bool, string, int, int64等，而tag中带有",string"选项，那么这个字段在输出到JSON的时候会把该字段对应的值转换成JSON字符串

~~~go
type Server struct {
	// ID 不会导出到JSON中
	ID int `json:"-"`

	// ServerName2 的值会进行二次JSON编码
	ServerName  string `json:"serverName"`
	ServerName2 string `json:"serverName2,string"`

	// 如果 ServerIP 为空，则不输出到JSON串中
	ServerIP   string `json:"serverIP,omitempty"`
}

s := Server {
	ID:         1,
	ServerName:  `Go "1.0" `,
	ServerName2: `Go "1.10" `,
	ServerIP:   ``,
}
b, _ := json.Marshal(s)
os.Stdout.Write(b)

// {"serverName":"Go \"1.0\" ","serverName2":"\"Go \\\"1.10\\\" \""}
~~~

Marshal函数只有在转换成功的时候才会返回数据,但是我们应该注意下:

- JSON对象只支持string作为key，所以要编码一个map，那么必须是map[string]T这种类型(T是Go语言中任意的类型)
- Channel, complex和function是不能被编码成JSON的
- 嵌套的数据是不能编码的，不然会让JSON编码进入死循环
- 指针在编码的时候会输出指针指向的内容，而空指针会输出null

- 解析到interface

在Go中Interface{}可以用来存储任意数据类型的对象，这种数据结构正好用于存储解析的未知结构的json数据的结果。JSON包中采用map[string]interface{}和[]interface{}结构来存储任意的JSON对象和数组。Go类型和JSON类型的对应关系如下：

* bool 代表 JSON booleans,
* loat64 代表 JSON numbers,
* string 代表 JSON strings,
* nil 代表 JSON null.

~~~go
// 通常情况下我们会拿到一段json数据:
b := []byte(`{"Name":"Ande","Age":10,"Hobby":"Football"}`)
// 现在开始解析到接口中:
var f interface{}
err := json.Unmarshal(b, &f)
//在这个接口f里面存储了一个map类型，他们的key是string，值存储在空的interface{}里
f = map[string]interface{}{
	"Name": "Ande",
	"Age":  10,
	"Hobby":"Football"
}
// 通过断言的方式我们把结构体强制转换数据类型：
m := f.(map[string]interface{})
// 通过断言之后，我们就可以通过来访问里面的数据：
for k, v := range m {
	switch vv := v.(type) {
	case string:
		fmt.Println(k, "is string", vv)
	case int:
		fmt.Println(k, "is int", vv)
	case float64:
		fmt.Println(k,"is float64",vv)
	case []interface{}:
		fmt.Println(k, "is an array:")
		for i, u := range vv {
			fmt.Println(i, u)
		}
	default:
		fmt.Println(k, "is of a type I don't know how to handle")
	}
}
~~~

通过interface{}与type assert的配合，我们就可以解析未知结构的JSON数了。

其实很多时候我们通过类型断言，操作起来不是很方便，我们可以使用一个叫做simplejson的包,在处理未知结构体的JSON数据:

~~~go
data,err := NewJson([]byte(`{                
	"test": {
		"array": [1, "2", 3, 4, 5]
		"int": 10,
		"float": 5.150,
		"bignum": 9223372036854775807,
		"string": "simplejson",
		"bool": true
	}
}`))

arr, _ := js.Get("test").Get("array").Array()
i, _ := js.Get("test").Get("int").Int()
ms := js.Get("test").Get("string").MustString()
~~~

#### json中的Encoder和Decoder

通常情况下，我们可能会从 Request 之类的输入流中直接读取 json 进行解析或将编码(encode)的 json 直接输出，为了方便，标准库为我们提供了 Decoder 和 Encoder 类型。它们分别通过一个 io.Reader 和 io.Writer 实例化，并从中读取数据或写数据。

~~~go
#Decoder 从 r io.Reader 中读取数据，`Decode(v interface{})` 方法把数据转换成对应的数据结构
func NewDecoder(r io.Reader) *Decoder

#Encoder 的 `Encode(v interface{})` 把数据结构转换成对应的 JSON 数据，然后写入到 w io.Writer 中
func NewEncoder(w io.Writer) *Encoder
~~~

#### json.RawMessage能够延迟对 json进行解码

此外我们在解析的时候，还可以把某部分先保留为 JSON 数据不要解析，等到后面得到更多信息的时候再去解析。