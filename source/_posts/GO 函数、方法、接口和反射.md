---
title: GO函数、方法、接口和反射
date: 2020/12/12
description: GO函数、方法、接口和反射
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/u9vgyrxMZbcAEG2.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/u9vgyrxMZbcAEG2.png'
categories:
  - go
tags:
  - go
  - 语法
abbrlink: 48362
---

# GO函数、方法、接口和反射

## 一、函数

Go是编译型语言，因此与函数编写的顺序是无关的，但是为了更好的可读性，需要把main() 函数写在文件的前面，其他函数按照一定逻辑顺序进行编写（例如函数被调用的顺序）。

编写多个函数的主要目的是将一个需要很多行代码的复杂问题分解为一系列简单的任务（那就是函数）来解决。而且，同一个任务（函数）可以被调用多次，有助于代码重用。

当函数执行到代码块最后一行（结束的}之前）或者 return 语句的时候会退出，其中 return 语句可以带有零个或多个参数，这些参数将作为返回值供调用者使用。**简单的 return 语句也可以用来结束 for 死循环，或者结束一个协程（goroutine）**。

在Go中主要有三种类型的函数：

1. 普通的带有名字的函数.
2. 匿名函数或者lambda函数.
3. 方法(Methods).

### 1. 函数参数与返回值

函数构成代码执行的逻辑结构。在Go语言中，函数的基本组成为：关键字 func 、函数名、参数列表、返回值、函数体和返回语句。除了main()、init()函数外，其它所有类型的函数都可以有参数与返回值。函数参数、返回值以及它们的类型被统称为函数签名。

~~~go
func function_name( [parameter list] ) [return_types] {
   body(函数体)
}
~~~

函数定义：

* func：函数由 func 开始声明
* function_name：函数名称，函数名和参数列表一起构成了函数签名。
* parameter list：参数列表，参数就像一个占位符，当函数被调用时，你可以将值传递给参数，这个值被称为实际参数。参数列表指定的是参数类型、顺序、及参数个数。参数是可选的，也就是说函数也可以不包含参数。
* return_types：返回类型，函数返回一列值。return_types 是该列值的数据类型。有些功能不需要返回值，这种情况下 return_types 不是必须的。
* body(函数体)：函数定义的代码集合。

形式参数列表描述了函数的参数名以及参数类型。这些参数作为局部变量，其值由参数调用者提供。返回值列表描述了函数返回值的变量名以及类型。如果函数返回一个无名变量或者没有返回值，返回值列表的括号是可以省略的。如果一个函数声明不包括返回值列表，那么函数体执行完毕后，不会返回任何值。

~~~go
// 这个函数计算两个int型输入数据的和，并返回int型的和
func plus(a int, b int) int {
	// Go需要使用return语句显式地返回值
	return a + b
}
fmt.Println(plus(3,4)) // 7
~~~

> 注意：GO中包内的函数，类型和变量的对外可见性(可访问性)由函数名，类型和变量标示符首字母决定，大写对外可见，小写对外不可见
> 概括来说: 公有函数的名字以大写字母开头；私有函数的名字以小写字母开头

你可能会偶尔遇到没有函数体的函数声明,这表示该函数不是以Go实现的。这样的声明定义了函数标识符。

~~~go
package math

func Sin(x float64) float //implemented in assembly language
~~~

~~~Go
package.Function(arg1, arg2, ..., argn)
~~~

Function 是 pack 包里面的一个函数，括号里的是被调用函数的实参（argument）：这些值被传递给被调用函数的形参（parameter）。函数被调用的时候，这些实参将被复制然后传递给被调用函数。函数一般是在其他函数里面被调用的，这个其他函数被称为调用函数（calling function）。函数能多次调用其他函数，这些被调用函数按顺序行，理论上，函数调用其他函数的次数是无穷的（直到函数调用栈被耗尽）。

在函数中有的时候可能也会遇到你要传递的参数的类型是多个(传递变长参数)如这样的函数:

~~~go
func patent(a,b,c ...int)
~~~

参数是采用 ...type 的形式传递，这样的函数称为变参函数.

### 2. 将函数作为参数传递

~~~go
package main

import (
    "fmt"
)

func main() {
    callback(1, Add)
}

func Add(a, b int) {
    fmt.Printf("The sum of %d and %d is: %d\n", a, b, a+b)
}

func callback(y int, f func(int, int)) {
    f(y, 5) // this becomes Add(1, 5)
}
~~~

### 3. 内置函数

Go语言拥有一些不需要进行导入操作就可以使用的内置函数。它们有时可以针对不同的类型进行操作，例如：len、cap 和 append，或必须用于系统级的操作，例如：panic。因而，它们需要直接获得编译器的支持。



| 名称               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| close              | 用于关闭管道通信channel                                      |
| len、cap           | len 用于返回某个类型的长度或数量（字符串、数组、切片、map 和管道）；cap 是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map） |
| new、make          | new 和 make 均是用于分配内存：new 用于值类型和用户定义的类型，如自定义结构,内建函数new分配了零值填充的元素类型的内存空间，并且返回其地址，一个指针类型的值。make 用于内置引用类型（切片、map 和管道）创建一个指定元素类型、长度和容量的slice。容量部分可以省略,在这种情况下,容量将等于长度。它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针）。它也可以被用于基本类型：v := new(int)。make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作,new() 是一个函数，不要忘记它的括号 |
| copy、append       | copy函数用于复制,copy返回拷贝的长度，会自动取最短的长度进行拷贝(min(len(src), len(dst))),append函数用于向slice追加元素 |
| panic、recover     | 两者均用于错误处理机制,使用panic抛出异常，抛出异常后将立即停止当前函数的执行并运行所有被defer的函数，然后将panic抛向上一层，直至程序carsh。recover的作用是捕获并返回panic提交的错误对象、调用panic抛出一个值、该值可以通过调用recover函数进行捕获。主要的区别是，即使当前goroutine处于panic状态，或当前goroutine中存在活动紧急情况，恢复调用仍可能无法检索这些活动紧急情况抛出的值。 |
| print、println     | 底层打印函数                                                 |
| complex、real imag | 用于创建和操作复数,imag返回complex的实部,real返回complex的虚部 |
| delete             | 从map中删除key对应的value                                    |

### 4. 递归函数

当一个函数在其函数体内调用自身，则称之为递归。递归是一种强有力的技术特别是在处理数据结构的过程中.

~~~go
package main

import "fmt"

func main() {
    result := 0
    for i := 0; i <= 10; i++ {
        result = processing(i)
        fmt.Printf("processing(%d) is: %d\n", i, result)
    }
}

func processing(n int) (res int) {
    if n <= 1 {
        res = 1
    } else {
        res = processing(n-1) + processing(n-2)
    }
    return
}
~~~

在使用递归函数时经常会遇到的一个重要问题就是栈溢出：一般出现在大量的递归调用导致的程序栈内存分配耗尽。这个问题可以通过一个名为懒惰求值的技术解决，在 Go 语言中，我们可以使用管道（channel）和 goroutine也会通过这个方案来优化斐波那契数列的生成问题。

这样许多问题都可以使用优雅的递归来解决，比如说著名的快速排序算法。

### 5. 匿名函数

当我们不希望给函数起名字的时候，可以使用匿名函数，例如：func(x, y int) int { return x + y }。
函数字面量的语法和函数声明相似,区别在于func关键字后没有函数名。函数值字面量是一种表达式,它的值被称为匿名函数(anonymous function)。匿名函数由一个不带函数名的函数声明和函数体组成.通常不希望再次使用(即只使用一次的)的函数可以定义为匿名函数.

~~~go
匿名函数结构:

func() {
    //func body
}() //花括号后加()表示函数调用，此处声明时为指定参数列表，
    
如:

fun(a,b int) {
   fmt.Println(a+b)
}(1,2) 	
~~~

goroutine是常见的匿名函数，通常我们会使用关键字 go 启动了一个匿名函数作为 goroutine。
使用匿名函数或闭包创建 goroutine 时，除了将函数定义部分写在 go 的后面之外，还需要加上匿名函数的调用参数，格式如下

~~~go
go func( 参数列表 ){
    函数体
}( 调用参数列表 )
~~~

* 参数列表：函数体内的参数变量列表。
* 函数体：匿名函数的代码。
* 调用参数列表：启动 goroutine 时，需要向匿名函数传递的调用参数。

~~~go
var x int64 = 20
var y int64 = 10
var wg sync.WaitGroup

wg.Add(1)
//定义一个匿名函数，并对该函数开启协程
go func(x, y int64) {
	  z := x+y
	  fmt.Println("the reuslt value:",z)
	  wg.Done()
}(x,y)
//由于这个函数是匿名函数，所以调用方式就直接是（x,y）去调用，不用输入函数名。

wg.Wait()
~~~

Goroutine是异步执行的，有的时候为了防止在结束mian函数的时候结束掉Goroutine，所以需要同步等待，这个时候就需要用 WaitGroup了，在 sync 包中，提供了 WaitGroup ，它会等待它收集的所有 goroutine 任务全部完成。在WaitGroup里主要有三个方法:

- Add：可以添加或减少 goroutine的数量
- Done：相当于Add(-1)
- Wait： 执行后会堵塞主线程，直到WaitGroup 里的值减至0

### 6. defer 延迟函数

Go语言的defer算是一个语言的新特性，至少对比当今主流编程语言如此。

**defer语句调用一个函数，这个函数执行会推迟，直到外围的函数返回，或者外围函数运行到最后，或者相应的goroutine panic**

defer语句经常被用于处理成对的操作：如打开、关闭、连接、断开连接、加锁、释放锁。通过defer机制，不论函数逻辑多复杂，都能保证在任何执行路径下，资源被释放。释放资源的defer应该直接跟在请求资源的语句后。

~~~go
f,err := os.Open(filename)
if err != nil {
    panic(err)
}
defer f.Close()
~~~

**如果有多个defer表达式，调用顺序类似于栈，越后面的defer表达式越先被调用。**

defer确实是在return之前调用的。但表现形式上却可能不像。
本质原因是return A语句并不是一条原子指令，defer被插入到了赋值 与ret之间，因此可能有机会改变最终的返回值。

### 7. panic 异常

错误和异常这两个是不同的概念，非常容易混淆。很多人习惯将一切非正常情况都看做错误，而不区分错误和异常，即使程序中可能有异常抛出，也将异常及时捕获并转换成错误。错误指的是可能出现问题的地方出现了问题，比如压缩一个文件时失败，这种情况在人们可以意料之中的事情；但是异常指的是不应该出现问题的地方出现了问题，比如引用了空指针，这种情况在人们的意料之外。因而，错误是业务过程的一部分，而异常不是。

Golang中引入error接口类型作为错误处理的标准模式，如果函数要返回错误，则返回值类型列表中肯定包含error。error处理过程类似于C语言中的错误码，可逐层返回，直到被处理。

Golang中引入两个内置函数panic和recover来触发和终止异常处理流程，同时引入关键字defer来延迟执行defer后面的函数。

一直等到包含defer语句的函数执行完毕时，延迟函数（defer后的函数）才会被执行，而不管包含defer语句的函数是通过return的正常结束，还是由于panic导致的异常结束。你可以在一个函数中执行多条defer语句，它们的执行顺序与声明顺序相反。

当程序运行时候，如果遇到引用空指针、下标越界或显式调用panic函数等情况，则会先触发panic函数的执行，然后调用延迟函数。调用者继续传递panic，因此该过程一直在调用栈中重复发生：函数停止执行，调用延迟执行函数等。如果一路在defer延迟函数中没有recover函数的调用，则会到达协程的起点，该协程结束，然后终止其他所有协程，包括主协程.

错误和异常从Golang机制上讲，就是error和panic的区别。

一般而言,当panic异常发生时,程序会中断运行,并立即执行在该goroutine中被延迟的函数(defer 机制)。随后,程序崩溃并输出日志信
息。日志信息包括panic value和函数调用的堆栈跟踪信息。panic value通常是某种错误信息。对于每个goroutine,日志信息中都会有与之相对的,发生panic时的函数调用堆栈跟踪信息。通常,我们不需要再次运行程序去定位问题,日志信息已经提供了足够的诊断依据。因此,在我们填写问题报告时,一般会将panic异常和日志信息一并记录。

虽然Go的panic机制类似于其他语言的异常,但panic的适用场景有一些不同。由于panic会引起程序的崩溃,因此panic一般用于严重错误,如程序内部的逻辑不一致。通常认为任何崩溃都表明代码中存在漏洞,所以对于大部分漏洞,我们应该使用Go提供的错误机制,而不是panic,尽量避免程序的崩溃。在健壮的程序中,任何可以预料到的错误,如不正确的输入、错误的配置或是失败的I/O操作都应该被优雅的处理,最好的处理方式,就是使用Go的错误机制。

### 8. recover捕获异常

通常来说，不应该对panic异常做任何处理。但有时，也许我们可以从异常中恢复，至少我们可以在程序崩溃前做一些操作。如果在deferred函数中调用了内置函数recover，并且定义该defer语句的函数发生了panic异常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运行，但能正常返回。在未发生panic时调用recover，recover会返回nil。

~~~go
func recover() interface{}
~~~

recover 函数用来获取 panic 函数的参数信息，只能在延时调用 defer 语句调用的函数中直接调用才能生效，如果在 defer 语句中也调用 panic 函数，则只有最后一个被调用的 panic 函数的参数会被 recover 函数获取到。如果 goroutine 没有 panic，那调用 recover 函数会返回 nil。

让我们以语言解析器为例，说明recover的使用场景。考虑到语言解析器的复杂性，即使某个语言解析器目前工作正常，也无法肯定它没有漏洞。因此,当某个异常出现时，我们不会选择让解析器崩溃，而是会将panic异常当作普通的解析错误，并附加额外信息提醒用户报告此错误。

~~~go
func Parse(input string) (s *Syntax, err error) {
  defer func() {
    if p := recover(); p != nil {
      err = fmt.Errorf("internal error: %v", p)
    }
 }()
 // ...parser...
}
~~~

从中可以看到defer函数帮助Parse从panic中恢复。在defer函数内部，panic value被附加到错误信息中；并用err变量接收错误信息，返回给调用者。我们也可以通过调用runtime.Stack往错误信息中添加完整的堆栈调用信息。

但是如果不加区分的恢复所有的panic异常，不是可取的做法；因为在panic之后，无法保证包级变量的状态仍然和我们预期一致。比如，对数据结构的一次重要更新没有被完整完成、文件或者网络连接没有被关闭、获得的锁没有被释放。此外，如果写日志时产生的panic被不加区分的恢复，可能会导致漏洞被忽略。

虽然把对panic的处理都集中在一个包下，有助于简化对复杂和不可以预料问题的处理，但作为被广泛遵守的规范，你不应该试图去恢复其他包引起的panic。公有的API应该将函数的运行失败作为error返回，而不是panic。同样的，你也不应该恢复一个由他人开发的函数引起的panic，比如说调用者传入的回调函数，因为你无法确保这样做是安全的。

因此安全的做法是有选择性的recover。换句话说，只恢复应该被恢复的panic异常。此外这些异常所占的比例应该尽可能的低。为了标识某个panic是否应该被恢复，我们可以将panic value设置成特殊类型。在recover时对panic value进行检查，如果发现panic value是特殊类型，就将这个panic作为errror处理。如果不是则按照正常的panic进行处理.

## 二、 方法

### 1. 一般方法

Golang方法的是作用在接收者(receiver)上的一个函数，接收者是某种类型的变量，因此Golang方法是一种特殊类型的函数。

接收者类型可以是（几乎）任何类型，不仅仅是结构体类型：任何类型都可以有方法，甚至可以是函数类型，可以是 `int`、`bool`、`string` 或数组的别名类型。但是接收者不能是一个接口类型，因为接口是一个抽象定义，但是方法却是具体实现；如果这样做会引发一个编译错误：`invalid receiver type…`。

最后接收者不能是一个指针类型，但是它可以是任何其他允许类型的指针。

对于方法来说，最简单的解释就是，在函数声明时，在其名字之前放上一个变量即是一个方法。这个附加的参数会将该函数附加到这种类型上，即相当于为这种类型定义了一个独占的方法。

~~~go
 package geometry
 import "math"
 type Point struct{ X, Y float64 }
  // traditional function
 func Distance(p, q Point) float64 {
   return math.Hypot(q.X-p.X, q.Y-p.Y)
 }
 // same thing, but as a method of the Point type
 func (p Point) Distance(q Point) float64 {
  return math.Hypot(q.X-p.X, q.Y-p.Y) 
 }
~~~

在Go语言中，我们并不会像其它语言那样用this或者self作为接收器，我们可以任意的选择接收器的名字。由于接收器的名字经常会被使用到，所以保持其在方法间传递时的一致性和简短性是不错的主意。这里的建议是可以使用其类型的第一个字母，比如这里使用了Point的首字母p。

在方法调用过程中，接收器参数一般会在方法名之前出现。这和方法声明是一样的，都是接收器参数在方法名字之前。

~~~go
p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q)) // "5", function call
fmt.Println(p.Distance(q))  // "5", method call
~~~

可以看到,上面的两个函数调用都是Distance，但是却没有发生冲突。第一个Distance的调用实际上用的是包级别的函数geometry.Distance，而第二个则是使用刚刚声明的Point，调用的是Point类下声明的Point.Distance方法。

这种p.Distance的表达式叫做选择器，因为他会选择合适的对应p这个对象的Distance方法来执行。选择器也会被用来选择一个struct类型的字段，比如p.X。由于方法和字段都是在同一命名空间，所以如果我们在这里声明一个X方法的话，编译器会报错。因为在调用p.X时会有歧义。

因为每种类型都有其方法的命名空间，我们在用Distance这个名字的时候，不同的Distance调用指向了不同类型里的Distance方法。让我们来定义一个Path类型，这个Path代表一个线段的集合，并且也给这个Path定义一个叫Distance的方法。

~~~go
// A Path is a journey connecting the points with straight lines.
type Path []Point
// Distance returns the distance traveled along the path.
func (path Path) Distance() float64 {
    sum := 0.0
    for i := range path {
     if i > 0 {
        sum += path[i-1].Distance(path[i])
     }
}
  return sum
}
~~~

这里的Path是一个命名的slice类型，而不是Point那样的struct类型，然而我们依然可以为它定义方法。在能够给任意类型定义方法这一点上，Go和很多其它的面向对象的语言不太一样。因此在Go语言里，我们为一些简单的数值、字符串、slice、map来定义一些附加行为很方便。方法可以被声明到任意类型，只要不是一个指针或者一个interface。

两个Distance方法有不同的类型。他们两个方法之间没有任何关系，尽管Path的Distance方法会在内部调用Point.Distance方法来计算每个连接邻接点的线段的长度。

### 2. 基于指针对象的方法

当调用一个函数时，会对其每一个参数值进行拷贝，如果一个函数需要更新一个变量，或者函数的其中一个参数实在太大我们希望能够避免进行这种默认的拷贝，这种情况下我们就需要用到指针了。对应到我们这里用来更新接收器的对象的方法，当这个接受者变量本身比较大时，我们就可以用其指针而不是对象来声明方法.

~~~go
func (p *Point) ScaleBy(factor float64) {
  p.X *= factor
  p.Y *= factor
}

r := &Point{1, 2}
r.ScaleBy(2)
fmt.Println(*r) // "{2, 4}"
~~~

需要注意的地方：

* 不管你的method的receiver是指针类型还是非指针类型,都是可以通过指针或者非指针类型进行调用的,编译器会帮你做类型转换。

* 在声明一个method的receiver该是指针还是非指针类型时,你需要考虑两方面的内部,第一方面是这个对象本身是不是特别大,如果声明为非指针变量时,调用会产生一次拷贝;第二方面是如果你用指针类型作为receiver,那么你一定要注意,这种指针类型指向的始终是一块内存地址,就算你对其进行了拷贝。

## 三、 接口

在Go语言中，函数和方法不太一样，有明确的概念区分。其他语言中，比如Java，一般来说，函数就是方法，方法就是函数，但是在Go语言中，
函数是指不属于任何结构体、类型的方法，也就是说，函数是没有接收者的；而方法是有接收者的，我们说的方法要么是属于一个结构体的，要么属于一个新定义的类型的。

在 Golang 中，interface 是一种抽象类型，相对于抽象类型的是具体类型（concrete type）：int，string。如下是 io 包里面的例子。

在 Golang中，interface是一组 method 的集合，是 duck-type programming 的一种体现。不关心属性（数据），只关心行为（方法）。具体使用中你可以自定义自己的 struct，并提供特定的 interface 里面的 method 就可以把它当成 interface 来使用。下面是一种 interface 的典型用法，定义函数的时候参数定义成 interface，调用函数的时候就可以做到非常的灵活。

~~~go
type FirstInterface interface{
    Print()
}

func TestFunc(x FirstInterface) {}
type PatentStruct struct {}
func (pt PatentStruct) Print() {}

func main() {
    var pt PatentStruct
    TestFunc(me)
}
~~~

* 泛型编程
* 隐藏具体实现
* 提供拦截端点

在 Golang 中并不支持泛型编程。在 C++ 等高级语言中使用泛型编程非常的简单，所以泛型编程一直是 Golang 诟病最多的地方。
但是使用 interface 我们可以实现泛型编程，我这里简单说一下，具体可以参考我前面给出来的那篇文章。比如我们现在要写一个泛型算法，形参定义采用 interface 就可以了，以标准库的 sort 为例。

~~~go
package sort

// A type, typically a collection, that satisfies sort.Interface can be
// sorted by the routines in this package.  The methods require that the
// elements of the collection be enumerated by an integer index.
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}

...

// Sort sorts data.
// It makes one call to data.Len to determine n, and O(n*log(n)) calls to
// data.Less and data.Swap. The sort is not guaranteed to be stable.
func Sort(data Interface) {
    // Switch to heapsort if depth of 2*ceil(lg(n+1)) is reached.
    n := data.Len()
    maxDepth := 0
    for i := n; i > 0; i >>= 1 {
        maxDepth++
    }
    maxDepth *= 2
    quickSort(data, 0, n, maxDepth)
}
~~~

Sort 函数的形参是一个 interface，包含了三个方法：Len()，Less(i,j int)，Swap(i, j int)。
使用的时候不管数组的元素类型是什么类型（int, float, string…），只要我们实现了这三个方法就可以使用 Sort 函数，这样就实现了“泛型编程”。
有一点比较麻烦的是，我们需要将数组自定义一下。下面是一个例子。

~~~go
type Person struct {
    Name string
    Age  int
}

func (p Person) String() string {
    return fmt.Sprintf("%s: %d", p.Name, p.Age)
}

// ByAge implements sort.Interface for []Person based on
// the Age field.
type ByAge []Person //自定义

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func main() {
    people := []Person{
        {"Bob", 31},
        {"John", 42},
        {"Michael", 17},
        {"Jenny", 26},
    }

    fmt.Println(people)
    sort.Sort(ByAge(people))
    fmt.Println(people)
}
~~~

## 四、多值返回

Go语言支持函数方法的多值返回，也就说我们定义的函数方法可以返回多个值，比如标准库里的很多方法，都是返回两个值，第一个是函数需要返回的值，第二个是出错时返回的错误信息，这种的好处，我们的出错异常信息再也不用像Java一样使用一个Exception这么重的方式表示了，非常简洁。

~~~go
func main() {
	file, err := os.Open("/usr/bin")
	if err != nil {
		log.Fatal(err)
		return
	}
	fmt.Println(file)
}

// 如果返回的值，我们不想使用，可以使用_进行忽略.
file, _ := os.Open("/usr/bin")

func add(a, b int) (int, error) {
	return a + b, nil
}
~~~

函数方法声明定义的时候，采用逗号分割，因为时多个返回，还要用括号括起来。返回的值还是使用return 关键字，以逗号分割，和返回的声明的顺序一致。

函数方法的参数，可以是任意多个，这种我们称之为可以变参数。

~~~go
func main() {
	print("a","b","c")
}
func print (a ...interface{}){
	for _,v:=range a{
		fmt.Print(v)
	}
	fmt.Println()
}

//  可变参数本质上是一个数组，所以我们向使用数组一样使用它，比如例子中的 for range 循环。
~~~

## 五、反射

在运行时反射是程序检查其所拥有的结构，尤其是类型的一种能力；这是元编程的一种形式。它同时也是造成迷惑的来源。反射可以在运行时检查类型和变量，例如它的大小、方法和 动态 的调用这些方法。这对于没有源代码的包尤其有用。

golang实现反射是通过reflect包来实现的, 让原本是静态类型的go具备了很多动态类型语言的特征。reflect包有两个数据类型，一个是Type,一个是Value。它定义了两个重要的类型,Type和Value. Type就是定义的类型的一个数据类型，Value是值的类型, TypeOf和ValueOf是获取Type和Value的方法。一个Type表示一个Go类型.它是一个接口, 有许多方法来区分类型以及检查它们的组成部分, 例如一个结构体的成员或一个函数的参数等. 唯一能反映 reflect.Type 实现的是接口的类型描述信息,也正是这个实体标识了接口值的动态类型.

接着我们开始我们使用Golang反射，通常在使用到Golang反射的时候会有三种定律:

### 1. 反射可以将“接口类型变量”转换为“反射类型对象”．

这里的反射类型指的是 reflect.Type 和 reflect.Value．将接口类型变量转换为反射类型变量，是通过 reflect 包的 **TypeOf** 和 **ValueOf** 实现的。

下面我们来看看在reflect包中的 **TypeOf** 和 **ValueOf** 的定义：

~~~go
#TypeOf returns the reflection Type that represents the dynamic type of i. If i is a nil interface value, TypeOf returns nil.
#TypeOf返回表示i的动态类型的反射Type。如果i是nil接口值，则TypeOf返回nil。

func TypeOf(i interface{}) Type

# ValueOf returns a new Value initialized to the concrete value stored in the interface i. ValueOf(nil) returns the zero Value
# ValueOf返回一个新的Value，初始化为存储在接口i中的具体值。 ValueOf（nil）返回零值

func ValueOf(i interface{}) Value
~~~

然后我们可以使用reflect.ValueOf和reflect.TypeOf将接口类型变量分别转换为反射类型：

~~~go
var p int = 10
v1 := reflect.ValueOf(p)//返回Value类型对象，值为10

t1 := reflect.TypeOf(p)//返回Type类型对象，值为int

fmt.Println("v1:",v1)

fmt.Println("t1",t1)

v2 := reflect.ValueOf(&p)//返回Value类型对象，值为&p，变量p的地址

t2 := reflect.TypeOf(&p)//返回Type类型对象，值为*int

fmt.Println("v2:",v2)

fmt.Println("t2:",t2)

// v1: 10
// t1: int
// v2: 0xc4200180b8
// t2: *int
~~~

其中v1和v2中包含了接口中的实际值，t1和t2中包含了接口中的实际类型。由于 **reflect.ValueOf** 和 **reflect.TypeOf** 的参数类型都是interface{}，空接口类型，而返回值的类型是reflect.Value和reflect.Type，中间的转换由reflect包来实现。所以就实现了接口类型变量到反射类型对象的转换。

### 2. 反射可以将“反射类型对象”转换为“接口类型变量”

这里根据一个 reflect.Value类型的变量，我们可以使用Interface方法恢复其接口类型的值。事实上，这个方法会把 type和value信息打包并填充到一个接口变量中，然后返回。

~~~go
func (v Value) Interface() (i interface{})

// Interface returns v's current value as an interface{}. It is equivalent to:
// 接口将v的当前值作为接口{}返回。它相当于：

var i interface{} = (v's underlying value)

// It panics if the Value was obtained by accessing unexported struct fields.
// 如果通过访问未导出的struct字段获得Value，则会发生混乱。
~~~

然后我们可以使用Value将反射类型转换为接口类型变量：

~~~go
var a int = 10

v1 := reflect.ValueOf(&a) //返回Value类型对象，值为&a，变量a的地址

t1 := reflect.TypeOf(&a)  //返回Type类型对象，值为*int

fmt.Println("v1:",v1)

fmt.Println("t1:",t1)

v2 := v1.Interface() //返回空接口变量

v3 := v2.(*int)     //类型断言，断定v1中type=*int

fmt.Printf("%T %v\n", v3, v3)

fmt.Println("v3:",*v3)

// v1: 0xc420082010
// t1: *int
// *int 0xc420082010
// v3: 10
~~~

reflect.ValueOf 的逆操作是 reflect.Value.Interface方法.它返回一个 interface{}类型，装载着与reflect.Value相同的具体值,这样我们就可以将“反射类型对象”转换为“接口类型变量．

其实reflect.Value 和 interface{} 都能装载任意的值. 有所不同的是, 一个空的接口隐藏了值内部的表示方式和所有方法, 因此只有我们知道具体的动态类型才能使用类型断言来访问内部的值(就像上面那样), 内部值我们没法访问. 相比之下, 一个 Value 则有很多方法来检查其内容, 无论它的具体类型是什么.

### 3. 如果要修改反射类型对象，其值必须是“addressable”

在上面第一种反射定律将“接口类型变量”转换为“反射类型对象”我们可以知道，反射对象包含了接口变量中存储的值以及类型。如果反射对象中包含的值是原始值，那么可以通过反射对象修改原始值，如果反射对象中包含的值不是原始值（反射对象包含的是副本值或指向原始值的地址），那么该反射对象是不可以修改的。

那么我们可以通过CanSet函数可以判定反射对象是否可以修改。

~~~go
# CanSet reports whether the value of v can be changed. A Value can be changed only if it is addressable and was not obtained by the use of unexported struct fields. If CanSet returns false, calling Set or any type-specific setter (e.g., SetBool, SetInt) will panic.

# CanSet报告是否可以更改v的值.仅当值可寻址且未通过使用未导出的struct字段获取时，才能更改值。如果CanSet返回false，则调用Set或任何特定于类型的setter（例如，SetBool，SetInt）将会发生混乱。

func (v Value) CanSet() bool
~~~

注意:CanSet返回false,值是不可以修改的，如果返回true则是可以修改的。

~~~go
var p float64 = 3.4

v1 := reflect.ValueOf(&p)

if v1.Kind() == reflect.Ptr && !v1.Elem().CanSet() { //判断是否为指针类型,元素是否可以修改
　　　fmt.Println("cannot set value")
　　　return
} else {
　　　v1 = v1.Elem() //实际取得的对象
　　　fmt.Println("CanSet return bool:", v1.CanSet())
}

v1.SetFloat(6.1)

fmt.Println("p=",p)

// CanSet return bool: true
// p = 6.1
~~~

从运行结果看值修改成功了，但是这里出现了Elem函数作用是用来获取原始值对应的反射对象。

~~~go
# Elem returns the value that the interface v contains or that the pointer v points to. It panics if v's Kind is not Interface or Ptr. It returns the zero Value if v is nil.
# Elem返回接口v包含的值或指针v指向的值。如果v的Kind不是Interface或Ptr，它会发生恐慌。如果v为nil，则返回零值。

func (v Value) Elem() Value
~~~

在这里要修改变量p的值，首先就要通过reflect.ValueOf来获取p的值类型,refelct.ValueOf返回的值类型是变量p一份值拷贝,要修改变量p就要传递p的地址,从而返回p的地址对象,才可以进行对p变量值对修改操作。在得到p变量的地址值类型之后先调用Elem()返回地址指针指向的值的Value封装。然后通过Set方法进行修改赋值。

通过反射可以很容易的修改变量的值，我们首先要通过反射拿到这个字段的地址值类型,然后去判断反射返回类型是否为reflect.Ptr指针类型（通过指针才能操作对象地址中的值)同时还要判断这个元素是否可以修改,通过Kind()和Set来修改字段的值,然后就可以拿到我们修改的值了．

因此在反射中使用反射包提供refelct.TypeOf和refelct.ValueOf方法获得接口对象的类型，值和方法等。通过反射修改字段值等时候需要传入地址类型，并且需要检查反射返回值类型是否为refelct.Ptr，检查字段是否CanSet,检查字段是存在,然后通过Kind()来帮助赋值相应对类型值。

~~~go
package main

import (
	"fmt"
	"reflect"
)

type User struct {
     Name   string `json:"name"`
     Gender string `json:"gender"`
     Age    int    `json:"age"`
}

func main()  {
     types := reflect.TypeOf(&User{}).Elem()
     value := reflect.ValueOf(&User{}).Elem()
     fmt.Println("values Numfield:",value.NumField())
     for i:=0;i<types.NumField();i++{
	m := types.Field(i).Tag.Get("json")
	fmt.Println(m)
     }
}

// values Numfield: 3
// name
// gender
// age
~~~

通常我们用反射修改一个变量的值:

~~~go
func main() {
	i := 1
	v := reflect.ValueOf(&i)
	v.Elem().SetInt(10)
	fmt.Println(i)
}

> go run reflect.go
10
~~~

1. 调用 reflect.ValueOf 函数获取变量指针；

2. 调用 reflect.Value.Elem 方法获取指针指向的变量；

3. 调用 reflect.Value.SetInt 方法更新变量的值；

由于 Go 语言的函数调用都是值传递的，所以我们只能先获取指针对应的 reflect.Value，再通过 reflect.Value.Elem 方法迂回的方式得到可以被设置的变量，我们通过如下所示的代码理解这个过程：

~~~go
func main() {
	i := 1
	v := &i
	*v = 10
}
~~~

如果不能直接操作 i 变量修改其持有的值，我们就只能获取 i 变量所在地址并使用 `*v` 修改所在地址中存储的整数。

### 反射的性能测试

Golang提供了一个testing包，使得单元测试、性能测试尤为简单。只要新建一个以_test结尾的文件，然后使用命令go test就可以自动执行文件中的相应测试函数了（单元测试函数以Test开头，性能测试函数以Benchmark开头）。我们可以使用golang testing来做一下reflect的最简单的性能测试。

Type：Type类型用来表示一个go类型。

不是所有go类型的Type值都能使用所有方法。请参见每个方法的文档获取使用限制。在调用有分类限定的方法时，应先使用Kind方法获知类型的分类。调用该分类不支持的方法会导致运行时的panic。

reflect.Type中方法:

~~~go
// 通用方法
func (t *rtype) String() string // 获取 t 类型的字符串描述，不要通过 String 来判断两种类型是否一致。

func (t *rtype) Name() string // 获取 t 类型在其包中定义的名称，未命名类型则返回空字符串。

func (t *rtype) PkgPath() string // 获取 t 类型所在包的名称，未命名类型则返回空字符串。

func (t *rtype) Kind() reflect.Kind // 获取 t 类型的类别。

func (t *rtype) Size() uintptr // 获取 t 类型的值在分配内存时的大小，功能和 unsafe.SizeOf 一样。

func (t *rtype) Align() int  // 获取 t 类型的值在分配内存时的字节对齐值。

func (t *rtype) FieldAlign() int  // 获取 t 类型的值作为结构体字段时的字节对齐值。

func (t *rtype) NumMethod() int  // 获取 t 类型的方法数量。

func (t *rtype) Method() reflect.Method  // 根据索引获取 t 类型的方法，如果方法不存在，则 panic。

// 如果 t 是一个实际的类型，则返回值的 Type 和 Func 字段会列出接收者。 如果 t 只是一个接口，则返回值的 Type 不列出接收者，Func 为空值。
func (t *rtype) MethodByName(string) (reflect.Method, bool) // 根据名称获取 t 类型的方法。

func (t *rtype) Implements(u reflect.Type) bool // 判断 t 类型是否实现了 u 接口。

func (t *rtype) ConvertibleTo(u reflect.Type) bool // 判断 t 类型的值可否转换为 u 类型。

func (t *rtype) AssignableTo(u reflect.Type) bool // 判断 t 类型的值可否赋值给 u 类型。

func (t *rtype) Comparable() bool // 判断 t 类型的值可否进行比较操作

// 注意对于：数组、切片、映射、通道、指针、接口 
func (t *rtype) Elem() reflect.Type // 获取元素类型、获取指针所指对象类型，获取接口的动态类型
// 数值
func (t *rtype) Bits() int  // 获取数值类型的位宽，t 必须是整型、浮点型、复数型
// 数组
func (t *rtype) Len() int  // 获取数组的元素个数
// 映射
func (t *rtype) Key() reflect.Type // 获取映射的键类型
// 通道
func (t *rtype) ChanDir() reflect.ChanDir // 获取通道的方向
// 结构体
func (t *rtype) NumField() int  // 获取字段数量

func (t *rtype) Field(int) reflect.StructField  // 根据索引获取字段

func (t *rtype) FieldByName(string) (reflect.StructField, bool)  // 根据名称获取字段

func (t *rtype) FieldByNameFunc(match func(string) bool) (reflect.StructField, bool)  // 根据指定的匹配函数 math 获取字段

func (t *rtype) FieldByIndex(index []int) reflect.StructField  // 根据索引链获取嵌套字段

// 函数
func (t *rtype) NumIn() int // 获取函数的参数数量

func (t *rtype) In(int) reflect.Type // 根据索引获取函数的参数信息

func (t *rtype) NumOut() int // 获取函数的返回值数量

func (t *rtype) Out(int) reflect.Type // 根据索引获取函数的返回值信息

func (t *rtype) IsVariadic() bool  // 判断函数是否具有可变参数。
// 如果有可变参数，则 t.In(t.NumIn()-1) 将返回一个切片。
~~~

reflect.Value方法:

reflect.Value.Kind()：获取变量类别，返回常量.

用于获取值方法：

~~~go
func (v Value) Int() int64 // 获取int类型值，如果 v 值不是有符号整型，则 panic。

func (v Value) Uint() uint64 // 获取unit类型的值，如果 v 值不是无符号整型（包括 uintptr），则 panic。

func (v Value) Float() float64 // 获取float类型的值，如果 v 值不是浮点型，则 panic。

func (v Value) Complex() complex128 // 获取复数类型的值，如果 v 值不是复数型，则 panic。

func (v Value) Bool() bool // 获取布尔类型的值，如果 v 值不是布尔型，则 panic。

func (v Value) Len() int // 获取 v 值的长度，v 值必须是字符串、数组、切片、映射、通道。

func (v Value) Cap() int  // 获取 v 值的容量，v 值必须是数值、切片、通道。

func (v Value) Index(i int) reflect.Value // 获取 v 值的第 i 个元素，v 值必须是字符串、数组、切片，i 不能超出范围。

func (v Value) Bytes() []byte // 获取字节类型的值，如果 v 值不是字节切片，则 panic。

func (v Value) Slice(i, j int) reflect.Value // 获取 v 值的切片，切片长度 = j - i，切片容量 = v.Cap() - i。
// v 必须是字符串、数值、切片，如果是数组则必须可寻址。i 不能超出范围。

func (v Value) Slice3(i, j, k int) reflect.Value  // 获取 v 值的切片，切片长度 = j - i，切片容量 = k - i。
// i、j、k 不能超出 v 的容量。i <= j <= k。
// v 必须是字符串、数值、切片，如果是数组则必须可寻址。i 不能超出范围。

func (v Value) MapIndex(key Value) reflect.Value // 根据 key 键获取 v 值的内容，v 值必须是映射。
// 如果指定的元素不存在，或 v 值是未初始化的映射，则返回零值（reflect.ValueOf(nil)）

func (v Value) MapKeys() []reflect.Value // 获取 v 值的所有键的无序列表，v 值必须是映射。
// 如果 v 值是未初始化的映射，则返回空列表。

func (v Value) OverflowInt(x int64) bool // 判断 x 是否超出 v 值的取值范围，v 值必须是有符号整型。

func (v Value) OverflowUint(x uint64) bool  // 判断 x 是否超出 v 值的取值范围，v 值必须是无符号整型。

func (v Value) OverflowFloat(x float64) bool  // 判断 x 是否超出 v 值的取值范围，v 值必须是浮点型。

func (v Value) OverflowComplex(x complex128) bool // 判断 x 是否超出 v 值的取值范围，v 值必须是复数型。
~~~

设置值方法：

~~~go
func (v Value) SetInt(x int64)  //设置int类型的值

func (v Value) SetUint(x uint64)  // 设置无符号整型的值

func (v Value) SetFloat(x float64) // 设置浮点类型的值

func (v Value) SetComplex(x complex128) //设置复数类型的值

func (v Value) SetBool(x bool) //设置布尔类型的值

func (v Value) SetString(x string) //设置字符串类型的值

func (v Value) SetLen(n int)  // 设置切片的长度，n 不能超出范围，不能为负数。

func (v Value) SetCap(n int) //设置切片的容量

func (v Value) SetBytes(x []byte) //设置字节类型的值

func (v Value) SetMapIndex(key, val reflect.Value) //设置map的key和value，前提必须是初始化以后，存在覆盖、不存在添加
~~~

其他的方法：

~~~go
// 结构体相关：
func (v Value) NumField() int // 获取结构体字段（成员）数量

func (v Value) Field(i int) reflect.Value  //根据索引获取结构体字段

func (v Value) FieldByIndex(index []int) reflect.Value // 根据索引链获取结构体嵌套字段

func (v Value) FieldByName(string) reflect.Value // 根据名称获取结构体的字段，不存在返回reflect.ValueOf(nil)

func (v Value) FieldByNameFunc(match func(string) bool) Value // 根据匹配函数 match 获取字段,如果没有匹配的字段，则返回零值（reflect.ValueOf(nil)）


// 通道相关：
func (v Value) Send(x reflect.Value)// 发送数据（会阻塞），v 值必须是可写通道。

func (v Value) Recv() (x reflect.Value, ok bool) // 接收数据（会阻塞），v 值必须是可读通道。

func (v Value) TrySend(x reflect.Value) bool // 尝试发送数据（不会阻塞），v 值必须是可写通道。

func (v Value) TryRecv() (x reflect.Value, ok bool) // 尝试接收数据（不会阻塞），v 值必须是可读通道。

func (v Value) Close() // 关闭通道

// 函数相关
func (v Value) Call(in []Value) (r []Value) // 通过参数列表 in 调用 v 值所代表的函数（或方法）。函数的返回值存入 r 中返回。
// 要传入多少参数就在 in 中存入多少元素。
// Call 即可以调用定参函数（参数数量固定），也可以调用变参函数（参数数量可变）。

func (v Value) CallSlice(in []Value) []Value // 调用变参函数
~~~

