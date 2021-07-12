---
title: Go 环境变量、编程标准、程序测试和基础包
date: 2020/12/14
description: Go 环境变量、编程标准、程序测试和基础包
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/u9vgyrxMZbcAEG2.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/u9vgyrxMZbcAEG2.png'
categories:
  - go
tags:
  - go
  - 语法
abbrlink: 22341
---

# Go 环境变量、编程标准、程序测试和基础包

## 一、环境变量

Go语言各种值类型，包括字符串、整数、浮点数、布尔值等。下面是一些基本示例。

Go语言最主要的特性：

* 自动垃圾回收
* 更丰富的内置类型
* 函数多返回值
* 错误处理
* 匿名函数和闭包
* 类型和接口
* 并发编程
* 反射
* 语言交互性

接着熟悉下Go环境变量:

~~~bash
> go env
GO111MODULE="auto"
GOARCH="amd64"
GOBIN="/Users/admin/goTest/bin"
GOCACHE="/Users/admin/Library/Caches/go-build"
GOENV="/Users/admin/Library/Application Support/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOINSECURE=""
GOMODCACHE="/Users/admin/goTest/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="darwin"
GOPATH="/Users/admin/goTest"
GOPRIVATE=""
GOPROXY="https://goproxy.cn,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
GCCGO="gccgo"
AR="ar"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/kp/3yqnp9cj4f3_9539b06q4yyc0000gn/T/go-build238604917=/tmp/go-build -gno-record-gcc-switches -fno-common"
~~~

| 参数        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| GO111MODULE | 是一个环境变量, on 仍将强制使用Go模块,off 强制Go表现出GOPATH方式,auto 是默认模式。 |
| GOARCH      | 程序构建环境的目标计算架构                                   |
| GOBIN       | 该环境变量的值为 Go 程序的可执行文件的目录                   |
| GOCACHE     | 存储编译后信息的缓存目录                                     |
| GOENV       | 本地的go环境文件存储位置                                     |
| GOEXE       | 可执行文件名后缀(".exe" on Windows)                          |
| GOFLAGS     | 当给定标志被当前命令已知时，默认情况下以空格分隔的-flag = value设置列表将应用于go命令。 |
| GOHOSTARCH  | Go工具链二进制文件的体系结构（GOARCH）。                     |
| GOINSECURE  | 逗号分隔的模块路径前缀列表（按Go的path.Match语法），应始终以不安全的方式获取。仅适用于直接获取的依赖项。 |
| GOMODCACHE  | Go命令将存储下载的模块的目录。                               |
| GOOS        | 为其编译代码的操作系统。例如linux，darwin，windows，netbsd。 |
| GOPATH      | Go 命令依赖一个重要的环境变量,GOPATH允许多个目录，当有多个目录时，请注意分隔符. |
| GOPRIVATE   | Go的私有模块仓库                                             |
| GOPROXY     | Go的mod代理                                                  |
| GOROOT      | Go安装目录的绝对路径                                         |
| GOSUMDB     | 要使用的校验和数据库的名称，以及可选的公用密钥和URL。        |
| GOTMPDIR    | Go命令将在其中写入临时源文件，程序包和二进制文件的目录。     |
| GOTOOLDIR   | Go tool的安装目录                                            |
| GCCGO       | 为"go build -compiler = gccgo"运行的gccgo命令。              |
| AR          | 使用gccgo编译器进行构建时用于处理库归档文件的命令。          |
| CC          | 用于编译C代码                                                |
| CXX         | 用于编译C代码                                                |
| CGO_ENABLED | 是否支持cgo命令                                              |
| GOMOD       | 主模块go.mod的绝对路径。如果启用了模块感知模式，但没有go.mod，则GOMOD将为os.DevNull |

## 二、编程标准和规范

### 1. 行长

* 一行最长不超过80个字符，超过的使用换行展示，尽量保持格式优雅。

### 2. 文件名命名规范

* 文件名命名用小写，尽量见名思义，看见文件名就可以知道这个文件下的大概内容，对于源代码里的文件，文件名要很好的代表了一个模块实现的功能。

### 3. 包

* 包名应该为小写单词，不要使用下划线或者混合大小写。
* 每个包都应该有一个包注释，包如果有多个go文件，就只需要在入口文件写包注释.
* 概况以 Package 开头。

```go
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is Governed by a BSD-style
// license that can be found in the LICENSE file.
 
// Package strings implements simple functions to manipulate strings.
package strings
```

### 4. 命名

* 包名用小写,使用短命名,尽量和标准库不要冲突.
* 使用短命名，因为长名字并不会使得事物更易读，文档注释会比格外长的名字更有用.
* 需要导出的任何类型必须以大写字母开头.


### 5. 变量

* 全局变量：驼峰式，可导出的使用大写字母开头.
* 参数传递：驼峰式，小写字母开头.
* 局部变量：下划线风格命名.

### 6. 接口

* 单个函数的接口名以”er”作为后缀，如Reader,Writer
* 接口的实现则去掉“er”

```go
type Reader interface {
        Read(p ****byte) (n int, err error)
}
```

* 两个函数的接口名综合两个函数名

```go
type WriteFlusher interface {
    Write(****byte) (int, error)
    Flush error
}
```

* 三个以上函数的接口名，类似于结构体名

```go
type Vehicle interface {
    Start(****byte) 
    Stop error
    Recover
}
```

### 7. 函数和结构体

* 函数名采用驼峰命名法，尽量不要使用下划线
* 写概况，并且使用被声明的名字作为开头。

```go
// Compile parses a regular expression and returns, if successful, a Regexp
// object that can be used to match against text.
func Compile(str string) (regexp *Regexp, err error) {

// Request represents a request to run a command.
type Request struct { ...}
```

* 采用命名的多返回值，在godoc生成的文档中，带有返回值的函数声明更利于理解。

```go
func nextInt(b ****byte, pos int) (value, nextPos int, err error) {}

```

### 8.import

* 对标准包，程序内部包，第三方包进行分组。

```go
import (
    "encoding/json"         //标准包
    "strings"

    "spike/models"      //内部包
    "spike/utils"

    "github.com/go-sql-driver/mysql"    //第三方包
)

```

* 引用包时不要使用相对路径。

```go
// 错误的做法
import “../net”
 
// 正确的做法
import “github.com/repo/proj/src/net”
```

### 9. 错误处理

* error作为函数的值返回,必须尽快对error进行处理
* 错误描述如果是英文必须为小写，不需要标点结尾
* 采用独立的错误流进行处理
* 不要在逻辑代码中使用panic

不推荐的方式：

```go
if err != nil {
    // error handling
} else {
    // normal code
}
```

推荐的方式:

```go
if err != nil {
    // error handling
    return // or continue, etc.
}
// normal code
```

如果返回值需要初始化，则采用下面的方式:

```go
x, err := f
if err != nil {
    // error handling
    return
}
// use x
```

* panic

在main包中只有当实在不可运行的情况采用panic，例如文件无法打开，数据库无法连接导致程序无法 正常运行，但是对于其他的package对外的接口不能有panic，只能在包内采用。 建议在main包中使用log.Fatal来记录错误，这样就可以由log来结束程序。

* Recover

recover用于捕获runtime的异常，禁止滥用recover，在开发测试阶段尽量不要用recover，recover一般放在你认为会有不可预期的异常的地方。

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func {
        if err := recover; err != nil {
            log.Println("work failed:", err)
        }
    }
    // do 函数可能会有不可预期的异常
    do(work)
}
```

* Defer

defer在函数return之前执行，对于一些资源的回收用defer是好的，但也禁止滥用defer，defer是需要消耗性能的,所以频繁调用的函数尽量不要使用defer。

```go
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close  // f.Close will run when we're finished.

    var result ****byte
    buf := make(****byte, 100)
    for {
        n, err := f.Read(buf**0:**)
        result = append(result, buf**0:n**...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}

```

### 10. 控制结构

* if 

if接受初始化语句，约定如下方式建立局部变量

```go
if err := file.Chmod(0664); err != nil {
    return err
}
```

* for

采用短声明建立局部变量.

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

* range

如果只需要第一项（key），就丢弃第二个：

```go
for key := range m {
    if key.expired {
        delete(m, key)
    }
}

```

如果只需要第二项，则把第一项置为下划线.

```go
sum := 0
for _, value := range array {
    sum += value
}
```


* switch

Go的switch比C更普遍.

````go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
````

没有自动转换，但可以用逗号分隔的列表呈现:

```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}

```

虽然它们在Go中几乎不像其他类似C语言那么常见，但可以使用break语句尽早终止切换:

```go
for n := 0; n < len(src); n += size {
    switch {
    case src**n** < sizeOne:
        if validateOnly {
            break
        }
        size = 1
        update(src**n**)

    case src**n** < sizeTwo:
        if n+1 >= len(src) {
            err = errShortInput
            break Loop
        }
        if validateOnly {
            break
        }
        size = 2
        update(src**n** + src**n+1**<<shift)
    }
}

```

这里是使用两个switch语句的字节片的:

```go
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b ****byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a**i** > b**i**:
            return 1
        case a**i** < b**i**:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}

```

使用类型断言的语法和关键字类型:

```go
var t interface{}
t = functionOfSomeType
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}

```

* return 

尽早return：一旦有错误发生，马上返回

```go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat
if err != nil {
    f.Close
    return err
}
codeUsing(f, d)

```

### 11.方法的接收器

* 名称 一般采用strcut的第一个字母且为小写，而不是this或者其他不符合的命名习惯

```go
type Buffer struct {
	buf       ****byte   // contents are the bytes buf**off : len(buf)**
	off       int      // read at &buf**off**, write at &buf**len(buf)**
	bootstrap **64**byte // memory to hold first slice; helps small buffers avoid allocation.
	lastRead  readOp   // last read operation, so that Unread* can work correctly.

	// FIXME: it would be advisable to align Buffer to cachelines to avoid false
	// sharing.
}

func (b *Buffer) Bytes ****byte { return b.buf**b.off:** }
```

* 如果接收者是map,slice或者chan，不要用指针传递

```go
//Map
package main

import (
    "fmt"
)

type mp map**string**string

func (m mp) Set(k, v string) {
    m**k** = v
}

func main {
    m := make(mp)
    m.Set("k", "v")
    fmt.Println(m)
}
```

```go
//Channel
package main

import (
    "fmt"
)

type ch chan interface{}

func (c ch) Push(i interface{}) {
    c <- i
}

func (c ch) Pop interface{} {
    return <-c
}

func main {
    c := make(ch, 1)
    c.Push("i")
    fmt.Println(c.Pop)
}

```

* 如果需要对slice进行修改，通过返回值的方式重新赋值

```go
//Slice
package main

import (
    "fmt"
)

type slice ****byte

func main {
    s := make(slice, 0)
    s = s.addOne(42)
    fmt.Println(s)
}

func (s slice) addOne(b byte) ****byte {
    return append(s, b)
}

```

* 如果接收者是含有sync.Mutex或者类似同步字段的结构体，必须使用指针传递避免复制

```go
package main

import (
    "sync"
)

type T struct {
    m sync.Mutex
}

func (t *T) lock {
    t.m.Lock
}

func main {
    t := new(T)
    t.lock
}
```

* 如果接收者是大的结构体或者数组，使用指针传递会更有效率。

```go
package main

import (
    "fmt"
)

type T struct {
    data **1024**byte
}

func (t *T) Get byte {
    return t.data**0**
}

func main {
    t := new(T)
    fmt.Println(t.Get)
}
```

* append

```go
var a, b ****int
b = append(b, a...)

```

* 使用strings.TrimPrefix 去掉前缀，strings.TrimSuffix去掉后缀

```go
var s1 = "a value"
var s2 = "a"
var s3 = strings.TrimPrefix(s1, s2)
```

```go
var s1 = "value"
var s2 = "e"
var s3 = strings.TrimSuffix(s1,s2)
```


### 12.使用工具检查你的代码

* **go fmt**(https://golang.org/pkg/fmt/)
* **go vet**(https://golang.org/cmd/vet/)
* **go simple**(https://github.com/dominikh/go-tools/tree/master/cmd/gosimple)
* **go lint**(https://github.com/golang/lint)
* **errcheck**(https://github.com/kisielk/errcheck)
* **misspell**(https://github.com/client9/misspell)

## 三、程序测试

Golang的单元测试采用内置的测试框架,通过引入testing包以及自带的go test命令来实现单元测试和性能测试，testing框架和其他语言中的测试框架类似，你可以基于这个框架写针对相应函数的测试用例，也可以基于该框架写相应的压力测试用例.

此外建议安装下gotests插件自动生成测试代码:

```go
go get -u -v github.com/cweill/gotests/...
```

### 编写测试用例

因为`go test`命令只能在一个相应的目录下执行所有文件，所以我们接下来新建一个项目目录`gotest`,这样我们所有的代码和测试代码都在这个目录下。

接下来我们在该目录下面创建两个文件：test.go和go_test.go

test.go:这个文件里面我们是创建了一个包，里面有一个函数实现了除法运算:

```Go
package gotest

import (
	"errors"
)

func Division(a, b float64) (float64, error) {
	if b == 0 {
		return 0, errors.New("除数不能为0")
	}

	return a / b, nil
}

```

go_test.go:这是我们的单元测试文件，但是记住下面的这些原则：

* 文件名必须是`_test.go`结尾的，这样在执行`go test`的时候才会执行到相应的代码
* 你必须import `testing`这个包
* 所有的测试用例函数必须是`Test`开头
* 测试用例会按照源代码中写的顺序依次执行
* 测试函数`TestXxx`的参数是`testing.T`，我们可以使用该类型来记录错误或者是测试状态
* 测试格式：`func TestXxx (t *testing.T)`,`Xxx`部分可以为任意的字母数字的组合，但是首字母不能是小写字母**a-z**，例如`Testintdiv`是错误的函数名。

- 函数中通过调用`testing.T`的`Error`, `Errorf`, `FailNow`, `Fatal`, `FatalIf`方法，说明测试不通过，调用`Log`方法用来记录测试的信息。

下面是我们的测试用例的代码：
	

```Go
package gotest

import (
	"testing"
)

func Test_Division_1(t *testing.T) {
	if i, e := Division(6, 2); i != 3 || e != nil { //try a unit test on function
		t.Error("除法函数测试没通过") // 如果不是如预期的那么就报错
	} else {
		t.Log("第一个测试通过了") 
	}
}

func Test_Division_2(t *testing.T) {
	t.Error("error")
}

```

我们在项目目录下面执行`go test`,就会显示如下信息：

~~~bash
--- FAIL: Test_Division_2 (0.00 seconds)
	go_test.go:16: error
FAIL
exit status 1
FAIL	go	0.013s

# 从这个结果显示测试没有通过，因为在第二个测试函数中我们写死了测试不通过的代码`t.Error`，那么我们的第一个函数执行的情况怎么样呢？默认情况下执行`go test`是不会显示测试通过的信息的，我们需要带上参数`go test -v`，这样就会显示如下信息：

=== RUN Test_Division_1
--- PASS: Test_Division_1 (0.00 seconds)
	go_test.go:11: 第一个测试通过了
=== RUN Test_Division_2
--- FAIL: Test_Division_2 (0.00 seconds)
	go_test.go:16: =error
FAIL
exit status 1
FAIL	go	0.012s
~~~



上面的输出详细的展示了这个测试的过程，我们看到测试函数1`Test_Division_1`测试通过，而测试函数2`Test_Division_2`测试失败了，最后得出结论测试不通过。接下来我们把测试函数2修改成如下代码：

```go
func Test_Division_2(t *testing.T) {
	if _, e := Division(6, 0); e == nil { //try a unit test on function
		t.Error("Division did not work as expected.") // 如果不是如预期的那么就报错
	} else {
		t.Log("one test passed.", e) //记录一些你期望记录的信息
	}
}	
```

然后我们执行`go test -v`，就显示如下信息，测试通过了：

~~~bash
=== RUN Test_Division_1
--- PASS: Test_Division_1 (0.00 seconds)
	go_test.go:11: 第一个测试通过了
=== RUN Test_Division_2
--- PASS: Test_Division_2 (0.00 seconds)
	go_test.go:20: one test passed. 除数不能为0
PASS
ok  	go	0.013s
~~~

### 编写压力测试

名称以 Benchmark 为名称前缀的函数，只能接受 *testing.B 的参数，这种测试函数是性能压力测试函数。

压力测试用来检测函数(方法）的性能，和编写单元功能测试的方法类似,此处不再赘述，但需要注意以下几点：

* 压力测试用例必须遵循如下格式，其中XXX可以是任意字母数字的组合，但是首字母不能是小写字母

```go
func BenchmarkXXX(b *testing.B) { ... }
```

* `go test`不会默认执行压力测试的函数，如果要执行压力测试需要带上参数`-test.bench`，语法:`-test.bench="test_name_regex"`,例如`go test -test.bench=".*"`表示测试全部的压力测试函数
* 在压力测试用例中,请记得在循环体内使用`testing.B.N`,以使测试可以正常的运行
* 文件名也必须以`_test.go`结尾

下面我们新建一个压力测试文件webbench_test.go，代码如下所示：

```go
package gotest

import (
	"testing"
)

func Benchmark_Division(b *testing.B) {
	for i := 0; i < b.N; i++ { //use b.N for looping 
		Division(4, 5)
	}
}

func Benchmark_TimeConsumingFunction(b *testing.B) {
	b.StopTimer //调用该函数停止压力测试的时间计数

	//做一些初始化的工作,例如读取文件数据,数据库连接之类的,
	//这样这些时间不影响我们测试函数本身的性能

	b.StartTimer //重新开始时间
	for i := 0; i < b.N; i++ {
		Division(4, 5)
	}
}

```

我们执行命令`go test webbench_test.go -test.bench=".*"`，可以看到如下结果：

```
Benchmark_Division-4   	                     500000000	      7.76 ns/op	     456 B/op	      14 allocs/op
Benchmark_TimeConsumingFunction-4            500000000	      7.80 ns/op	     224 B/op	       4 allocs/op
PASS
ok  	gotest	9.364s
```

上面的结果显示我们没有执行任何`TestXXX`的单元测试函数，显示的结果只执行了压力测试函数，第一条显示了`Benchmark_Division`执行了500000000次，每次的执行平均时间是7.76纳秒，第二条显示了`Benchmark_TimeConsumingFunction`执行了500000000，每次的平均执行时间是7.80纳秒。最后一条显示总共的执行时间。

通过上面对单元测试和压力测试的学习，我们可以看到`testing`包很轻量，编写单元测试和压力测试用例非常简单，配合内置的`go test`命令就可以非常方便的进行测试，这样在我们每次修改完代码,执行一下go test就可以简单的完成回归测试了。

## 四、基础包

* **archive**
  * **tar--tar包实现了tar格式压缩文件的存取**
  * **zip--zip包提供了zip档案文件的读写服务**
* **bufio--bufio包实现了带缓存的I/O操作**
* **builtin-builtin包为Go的预声明标识符提供了文档**
* **bytes--bytes包实现了操作****byte的常用函数**
* **compress**
  * **bzip2--bzip2包实现bzip2的解压缩** 
  * **flate--flate包实现了deflate压缩数据格式，参见RFC 1951**   
  * **gzip--gzip包实现了gzip格式压缩文件的读写，参见RFC 1952**   
  * **lzw--lzw包实现了Lempel-Ziv-Welch数据压缩格式**   
  * **zlib--zlib包实现了对zlib格式压缩数据的读写，参见RFC 1950**  
* **container**      
  * **heap--heap包提供了对任意类型（实现了heap.Interface接口）的堆操作** 
  * **list--list包实现了双向链表** 
  * **ring--ring实现了环形链表的操作**
* **context--context包定义了上下文类型,它跨API边界和进程之间传送截止时间，取消信号和其他请求范围的值**   
* **crypto--crypto包搜集了常用的密码（算法）常量.** 
  * **aes--aes包实现了AES加密算法.**      
  * **cipher--cipher包实现了多个标准的用于包装底层块加密算法的加密算法实现.**      
  * **des--des包实现了DES标准和TDEA算法.**      
  * **dsa--dsa包实现FIPS 186-3定义的数字签名算法.**      
  * **elliptic--elliptic包实现了几条覆盖素数有限域的标准椭圆曲线.**      
  * **hmac--hmac包实现了规定的HMAC加密哈希信息认证码.**      
  * **md5--md5包实现了MD5哈希算法.**      
  * **rand--rand包实现了用于加解密的更安全的随机数生成器.**      
  * **rc4--rc4包实现了RC4加密算法.**      
  * **rsa--rsa包实现了PKCS#1规定的RSA加密算法.**      
  * **sha1--sha1包实现了SHA1哈希算法，参见RFC 3174.**      
  * **sha256--sha256包实现了SHA224和SHA256哈希算法，参见FIPS 180-4**      
  * **sha512--sha512包实现了SHA384和SHA512哈希算法，参见FIPS 180-2**(.)      
  * **subtle--package subtle实现了在加密代码中常用的功能，但需要仔细考虑才能正确使用**      
  * **tls--tls包实现了TLS 1.2，细节参见RFC 5246**      
  * **x509--x509包解析X.509编码的证书和密钥**      
    * **pkix--pkix包提供了共享的、低层次的结构体，用于ASN.1解析和X.509证书、CRL、OCSP的序列化**      
* **database**   
  * **sql--sql包提供了通用的SQL（或类SQL）数据库接口**    
    * **driver--driver包定义了应被数据库驱动实现的接口，这些接口会被sql包使用**  
* **debug**
  * **dwarf--提供了对从可执行文件加载的DWARF调试信息的访问**       
  * **elf--实现了对ELF对象文件的访问**       
  * **gosym--访问Go语言二进制程序中的调试信息**
  * **macho-实现了Mach-O对象文件的访问**
  * **macho-实现了Mach-O对象文件的访问**
  * **pe-实现了对PE（Microsoft Windows Portable Executable）文件的访问**
  * **plan9obj--plan9obj包实现了对Plan 9 a.out对象文件的访问**
* **encoding--encoding包定义了供其它包使用的可以将数据在字节水平和文本表示之间转换的接口**
  * **ascii85--ascii85 包是对 ascii85 的数据编码的实现**     
  * **asn1--asn1包实现了DER编码的ASN.1数据结构的解析，参见ITU-T Rec X.690**     
  * **base32--base32包实现了RFC 4648规定的base32编码**     
  * **base64--base64实现了RFC 4648规定的base64编码**     
  * **binary--binary包实现了简单的数字与字节序列的转换以及变长值的编解码**     
  * **csv--csv读写逗号分隔值（csv）的文件**     
  * **gob--gob包管理gob流——在编码器（发送器）和解码器（接受器）之间交换的binary值**     
  * **hex--hex包实现了16进制字符表示的编解码**     
  * **json--json包实现了json对象的编解码，参见RFC 4627**     
  * **pem--pem包实现了PEM数据编码（源自保密增强邮件协议）**     
  * **xml--xml包实现了一个简单的XML 1.0解析器，它可以理解XML名称空间**  
* **errors--error包实现了用于错误处理的函数**        
* **expvar--expvar包提供了公共变量的标准接口，如服务的操作计数器**        
* **flag--flag 包实现命令行标签解析**        
* **fmt--fmt 包实现了格式化I/O函数，类似于C的 printf 和 scanf**
* **go**
  * **ast--ast包声明了用于展示Go包中的语法树类型** 
  * **build--build包提供了构建Go包的工具** 
  * **constant--constant包实现表示无类型Go常量及其相应操作的值** 
  * **doc--doc包从Go AST中提取源代码文档** 
  * **format--format包实现Go源的标准格式** 
  * **importer--importer包提供对导出数据导入程序的访问** 
  * **parser--parser包为Go源文件实现解析器** 
  * **printer--printer包实现AST节点的打印** 
  * **scanner--scanner包为Go源文本实现扫描程序** 
  * **token--token包表示Go语言的词法标记的常量和标记的基本操作** 
  * **types--types包声明数据类型并实现Go包类型检查的算法** 
* **hash--hash包提供hash函数的接口**
  * **adler32--adler32包实现了Adler-32校验和算法，参见RFC 1950**
  * **crc32--crc32包实现了32位循环冗余校验（CRC-32）的校验和算法**
  * **crc64--crc64包实现64位循环冗余校验或CRC-64校验**
  * **fnv--fnv包实现了FNV-1和FNV-1a（非加密hash函数）**
* **html--html包提供了用于转义和解转义HTML文本的函数**
  * **template--template包（html/template）实现了数据驱动的模板，用于生成可对抗代码注入的安全HTML输出**   
* **image--image实现了基本的2D图片库**      
  * **color--color包实现了基本的颜色库**     
    * **palette--palette包提供了标准的调色板**  
  * **draw--draw包提供组装图片的方法**      
  * **gif--gif包实现了GIF图片的解码**      
  * **jpeg--jpeg包实现了jpeg格式图像的编解码**      
  * **png--png包实现了PNG图像的编码和解码** 
* **index**          
  * **suffixarray--suffixarrayb包通过使用内存中的后缀树实现了对数级时间消耗的子字符串搜索**
* **io--io包为I/O原语提供了基础的接口**
  * **ioutil--ioutil包实现了一些I/O的工具函数**
* **log--log包实现了简单的日志服务**   
  * **syslog--syslog包提供一个简单的系统日志服务的接口**  
* **math(math--math包提供了基本常数和数学函数)**
  * **big--big包实现了（大数的）高精度运算.**
  * **cmplx--cmplx包为复数提供了基本的常量和数学函数**
  * **rand--rand包实现了伪随机数生成器**
* **mime--mime实现了MIME的部分规定**
  * **multipart--multipart实现了MIME的multipart解析，参见RFC 2046**
  * **quotedprintable--quotedprintable包实现RFC 2045指定的quoted-printable编码**
* **net--net包提供了可移植的网络I/O接口，包括TCP/IP、UDP、域名解析和Unix域socket**     
  * **http--http包提供了HTTP客户端和服务端的实现** 
    * **cgi--cgi包实现了RFC3875协议描述的CGI(公共网关接口)**
    * **cookiejar--cookiejar包实现了保管在内存中的符合RFC6265标准的http.CookieJar接口**
    * **fcgi--fcgi包实现了FastCGI协议**
    * **httptest--httptest包提供HTTP测试的单元工具**
    * **httptrace--httptrace包提供了跟踪HTTP客户端请求中的事件的机制**
    * **httputil--httputil包提供了HTTP公用函数，是对net/http包的更常见函数的补充**
    * **pprof--pprof包通过提供HTTP服务返回runtime的统计数据，这个数据是以pprof可视化工具规定的返回格式返回的**
  * **mail--mail包实现了解析邮件消息的功能**
  * **rpc--rpc包提供了一个方法来通过网络或者其他的I/O连接进入对象的外部方法**
    * **jsonrpc--jsonrpc包使用了rpc的包实现了一个JSON-RPC的客户端解码器和服务端的解码器**
  * **smtp--smtp包实现了简单邮件传输协议（SMTP），参见RFC 5321**    
  * **textproto--textproto实现了对基于文本的请求/回复协议的一般性支持，包括HTTP、NNTP和SMTP**
  * **url--url包解析URL并实现了查询的逸码，参见RFC 3986**
* **os--os包提供了操作系统函数的不依赖平台的接口**
  * **exec--exec包执行外部命令**     
  * **signal--signal包实现了对输入信号的访问**     
  * **user--user包允许通过名称或ID查询用户帐户**   
* **path--path包实现了对斜杠分隔的路径的实用操作函数**  
  * **filepath--filepath包实现了兼容各操作系统的文件路径的实用操作函数**   
* **plugin--plugin包实现了Go插件的加载和符号解析** 
* **reflect--reflect包实现了运行时反射，允许程序操作任意类型的对象**   
* **regexp--regexp包实现了正则表达式搜索**      
  * **syntax--syntax包将正则表达式解析为语法树**      
* **runtime--runtime包含与Go的运行时系统进行交互的操作，例如用于控制Go程的函数**      
  * **cgo--cgo包含有cgo工具生成的代码的运行时支持**      
  * **debug--debug包含有程序在运行时调试其自身的功能**      
  * **pprof--pprof包按照可视化工具pprof所要求的格式写出运行时分析数据**      
  * **race--race包实现了数据竞争检测逻辑**      
  * **trace--trace包实现了Go执行追踪**    
* **sort--sort包为切片及用户定义的集合的排序操作提供了原语**   
* **strconv--strconv包实现了基本数据类型和其字符串表示的相互转换**
* **strings--strings包完成对字符串的主要操作**
* **sync--sync包提供了互斥锁这类的基本的同步原语**    
  * **atomic--atomic包提供了底层的原子性内存原语，这对于同步算法的实现很有用**    
  * **Map--线程安全的map**    
  * **Mutex--互斥锁**    
  * **Cond--条件变量**    
  * **Pool--临时对象池**    
  * **Once--执行一次**    
  * **RWMutex--读写操作的互斥锁**    
  * **waitGroup--用于等待一组goroutine 结束**    
* **syscall--syscall包含低级操作系统原语的接口**    
* **testing--testing包为Go的自动测试提供支持**
  * **iotest--iotest包实现了主要用于读和写的测试**
  * **quick--quick包实现实用程序功能，以帮助进行黑盒测试**
* **text**     
  * **scanner--scanner包提供对utf-8文本的token扫描服务**
  * **tabwriter--tabwriter包实现了写入过滤器（tabwriter.Writer），可以将输入的缩进修正为正确的对齐文本**
  * **template--template包实现了数据驱动的用于生成文本输出的模板**
    * **parse--parse包为文本/模板和html /模板定义的模板构建解析树**
* **time--time包提供了时间的显示和测量用的函数**        
* **unicode--unicode 包提供了一些测试Unicode码点属性的数据和函数**    
  * **utf16--utf16包实现了对UTF-16序列的编码和解码**    
  * **utf8--utf8包实现支持UTF-8文本编码的函数和常量**   
* **unsafe--unsafe包含有关于Go程序类型安全的所有操作**      