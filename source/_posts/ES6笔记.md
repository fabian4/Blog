---
title: ES6笔记
date: 2020/6/6
description: ES6笔记
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/j9w7l6HToiJuavk.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/j9w7l6HToiJuavk.jpg'
categories:
  - JavaScript
tags:
  - 笔记
  - JavaScript
abbrlink: 42571
---

# 一、 块级作用域绑定

## var

JavaScript中，我们通常说的作用域是函数作用域，使用var声明的变量，无论是在代码的哪个地方声明的，都会提升到当前作用域的最顶部，这种行为叫做**变量提升（Hoisting）**

也就是说，如果在函数内部声明的变量，都会被提升到该函数开头，而在全局声明的变量，就会提升到全局作用域的顶部。

```javascript
function test() {
    console.log('1: ', a) //undefined
    if (false) {
      var a = 1
    }
    console.log('3: ', a) //undefined
}

test()
```

实际执行时，上面的代码中的变量a会提升到函数顶部声明，即使if语句的条件是false，也一样不影响a变量提升。

```javascript
function test() {
    var a
    //a声明没有赋值
    console.log('1: ', a) //undefined
    if (false) {
      a = 1
    }
    //a声明没有赋值
    console.log('3: ', a) //undefined
}
```

在函数嵌套函数的场景下，变量只会提升到最近的一个函数顶部，而不会提升到外部函数。

```javascript
//b提升到函数a顶部，但不会提升到函数test。
    function test() {
        function a() {
          if (false) {
            var b = 2
          }
        }
        console.log('b: ', b)
    }
    
    test() //b is not defined
```

如果a没有声明，那么就会报错，**没有声明和声明后没有赋值是不一样的**，这点一定要区分开，有助于我们找bug。

```javascript
 //a没有声明的情况
    a is not defined
```

## let

let和const都能够声明块级作用域，用法和var是类似的，let的特点是不会变量提升，而是被锁在当前块中。

一个非常简单的例子：

```javascript
    function test() {
        if(true) {
          console.log(a)//TDZ，俗称临时死区，用来描述变量不提升的现象
          let a = 1
        }
    }
    test()  // a is not defined

    function test() {
        if(true) {
          let a = 1
        }
        console.log(a)
    }    
    test() // a is not defined
```

唯一正确的使用方法：**先声明，再访问。**

```javascript
function test() {
        if(true) {
          let a = 1
          console.log(a)
        }
    }
    test() // 1
```

## const

声明常量，一旦声明，不可更改，而且常量必须初始化赋值。

```javascript
const type = "ACTION"
```

我们试试重新声明type，看看会报什么错：     

```javascript
const type = "ACTION"
type = 1
console.log(type) //"type" is read-only
    
const type = "ACTION"
let type = 1
console.log(type) //Duplicate declaration "type"
```

const虽然是常量，不允许修改默认赋值，但如果定义的是对象Object，那么可以修改对象内部的属性值包括新增删除键值对也是可以的。

```javascript
const type = {
      a: 1
    }
type.a = 2 //没有直接修改type的值，而是修改type.a的属性值，这是允许的。
console.log(type) // {a: 2}

type.b = 3 //拓展Object也是没有问题的
console.log(type) // {a: 2 , b: 3}
    
delete type.b=3 //删除整个键值对也OK的
console.log(type) // {a: 2}
    
//如果重新定义数据结构~常量的内存地址值发生改变,这个是不可行的。
type={}; //Assignment to constant variable.
type=[]; //Assignment to constant variable.
```

## const和let的异同点

- **相同点：**const和let都是在当前块内有效，执行到块外会被销毁，也不存在变量提升（TDZ），不能重复声明。
- **不同点：**const不能再赋值，let声明的变量可以重复赋值。

## 临时死区(TDZ)

上面我们也提到了TDZ的场景，那么，有什么用呢？答案就是没什么用。

临时死区的意思是在当前作用域的块内，在声明变量前的区域叫做临时死区。
​    

```javascript
if (true) {
      //这块区域是TDZ
      let a = 1
    }
```

### 块级作用域的使用场景

除了上面提到的常用声明方式，我们还可以在循环中使用，最出名的一道面试题：循环中定时器闭包的考题

在for循环中使用var声明的循环变量，会跳出循环体污染当前的函数。

```javascript
for(var i = 0; i < 5; i++) {
      setTimeout(() => {
        console.log(i) //5, 5, 5, 5, 5
      }, 0)
    }
    console.log(i) //5 i跳出循环体污染外部函数
    
    //将var改成let之后
    for(let i = 0; i < 5; i++) {
      setTimeout(() => {
        console.log(i) // 0,1,2,3,4
      }, 0)
    }
    console.log(i)//i is not defined i无法污染外部函数
```

### 在全局作用域声明

如果在全局作用域使用let或者const声明，当声明的变量本身就是全局属性，比如closed。只会覆盖该全局变量，而不会替换它。

```javascript
    window.closed = false
    let closed = true
    
    closed // true
    window.closed // false
```

### 最佳实践

在实际开发中，我们选择使用var、let还是const，取决于我们的变量是不是需要更新，通常我们希望变量保证不被恶意修改，而使用大量的const，在react中，props传递的对象是不可更改的，所以使用const声明，声明一个对象的时候，也推荐使用const，当你需要修改声明的变量值时，使用let，var能用的场景都可以使用let替代。

# 二、字符串和正则表达式

## 字符串

字符串（String）是`JavaScript`6大原始数据类型。其他几个分别是Boolean、Null、Undefined、Number、Symbol（es6新增）。

字符串类型在前端开发者，是使用最频繁的类型之一，网站上可见的各种文案，几乎都是字符串类型的数据。我们经常需要使用的操作无非是这么几点：读取字符串、转换字符串、清空字符串、拼接字符串、截取字符串。

在ES5中，字符串类型已经有了非常丰富的应用能力，那么，在ES6中，ECMA的专家们对字符串做了什么更新呢？

当Unicode引入扩展字符集之后，16位的字符已经不足以满足字符串的发展，所以才在ES6中更新了Unicode的支持。

我们看看ES6字符串新增的方法

**UTF-16码位：**ES6强制使用UTF-16字符串编码。关于UTF-16的解释请自行百度了解。

**codePointAt()：**
该方法支持UTF-16，接受编码单元的位置而非字符串位置作为参数，返回与字符串中给定位置对应的码位，即一个整数值。

**String.fromCodePoiont()：**作用与codePointAt相反，检索字符串中某个字符的码位，也可以根据指定的码位生成一个字符。

**normalize()**：提供Unicode的标准形式，接受一个可选的字符串参数，指明应用某种Unicode标准形式。

## 正则表达式

**正则表达式u修饰符：**
当给正则表达式添加u字符时，它就从编码单元操作模式切换为字符模式。

### 其他新增的方法

上面提到的字符串和正则的新增方法只有在国际化的时候才用的到，我想，国内的很多网站还是不需要考虑国际化的问题，看不懂就先丢掉。下面讲到的新增的方法是实际开发中需求比较频繁的方法。

**字符串中的子串识别**：

以前我们经常使用indexOf()来检测字符串中是否包含另外一段字符串。

```javascript
    let t = 'abcdefg'
    if(t.indexOf('cde') > -1) {
      console.log(2)
    }
    //输出2，因为t字符串中包含cde字符串。
```

在ES6中，新增了3个新方法。每个方法都接收2个参数，需要检测的子字符串，以及开始匹配的索引位置。

**includes(str, index)：**如果在字符串中检测到指定文本，返回true，否则false。

```javascript
    let t = 'abcdefg'
    if(t.includes('cde')) {
      console.log(2)
    }
    //true
```

**startsWith(str, index)**：如果在字符串起始部分检测到指定文本，返回true，否则返回false。

```javascript
    let t = 'abcdefg'
    if(t.startsWith('ab')) {
      console.log(2)
    }
    //true
```

**endsWith(str, index)**：如果在字符串的结束部分检测到指定文本，返回true，否则返回false。

```javascript
    let t = 'abcdefg'
    if(t.endsWith('fg')) {
      console.log(2)
    }
    //true
```

**如果你只是需要匹配字符串中是否包含某子字符串，那么推荐使用新增的方法，如果需要找到匹配字符串的位置，使用indexOf()。**

**repeat(number)**

这个方法挺有意思的，接收一个Number类型的数据，返回一个重复N次的新字符串。即使这个字符串是空字符，也你能返回N个空字符的新字符串。

```javascript
console.log('ba'.repeat(3)) //bababa
```

### 正则表达式的其他更新

正则表达式y修饰符、正则表达式的复制、flags属性......

由于这一块知识没用过，就不打算去研究实际用途。

### 模板字面量

以前，我们用单引号或双引号表示字符串。

```javascript
let a = '123' //单引号
let b = "123" //双引号
```

现在，使用模板字面量反撇号``。在实际开发中，这是经常都要用到的方法。

```javascript
let c = `123` //反撇号
```

在字符串中使用反撇号，只需要加上转义符。

```javascript
let d = `12\`3` //字符串内插入反撇号的方式。
```

**在多行字符串的使用价值：**

模板字面量为解决多行字符串的一系列问题提供了一个非常好的机制。

如果不使用模板字面量，实现多行字符串，你可能会使用换行符。

```javascript
    let a = '123\n456'
    console.log(a) 
    // 123
    // 456
```

使用模板字面量，就可以非常简单的实现需求。

```javascript
    let a = `123
    456
    `
    console.log(a)
    // 123
    // 456
```

**在模板字面量插入变量的方法。**

我们不再需要使用 +（加号）来向字符串插入变量，而是使用${params}直接插入你需要添加到字符串的位置。

```javascript
    let t = 'haha'
    let a = `123${t}456`
    console.log(a) //123haha456
```

这种方式也叫作字符串占位符。占位符支持互相嵌套模板字面量，强大吧。有了它，我们终于可以抛弃 + 拼接字符串的恶心做法了。

**模板字面量的终极用法**
tag是一个方法，方法名你可以任意命名，这种写法被称作标签模板。

```javascript
    function tag(literals, ...substitutions) {
        //literals是数组，第一个位置是""，第二个位置是占位符之间的字符串，在本例中是haha
        //substitutions是字符串中的模板字面量，可能多个
        
        //函数最终返回字符串
    }
    let a = 4
    let t = tag`${a} haha`
    console.log(t) //4 haha
```

# 三、函数

## 函数的默认参数

在ES5中，我们给函数传参数，然后在函数体内设置默认值，如下面这种方式。

```javascript
    function a(num, callback) {
      num = num || 6
      callback = callback || function (data) {console.log('ES5: ', data)}
      callback(num * num)
    }
    a() //ES5: 36，不传参输出默认值
    
    //你还可以这样使用callback
    a(10, function(data) {
      console.log(data * 10) // 1000， 传参输出新数值
    })
```

**而在ES6中，我们使用新的默认值写法。**

```javascript
    function a(num = 6, callback = function (data) {console.log('ES6: ', data)}) {
      callback(num * num)
    }
    
    a() //ES6: 36， 不传参输出默认值
    
    a(10, function(data) {
      console.log(data * 10) // 1000，传参输出新数值
    })
```

**使用ES6的默认值写法可以让函数体内部的代码更加简洁优雅**

**默认值对arguments对象的影响**

我们先要了解arguments对象是什么？准确一点来说它是一个类数组对象，它存在函数内部，它将当前函数的所有参数组成了一个类数组对象。

```javascript
    function a(num, b){
      console.log(arguments) // {"0": 6, "1": 10}
      console.log(arguments.length) // 2
    }
    
    a(6, 10) 
```

上面的输出结果看起来很正常，那么，如果我们加上参数默认值会怎样呢？

```javascript
    function a(num = 1, b = 1){
      console.log(arguments)
    }
    a() // {} 默认值不能被arguments识别。
    a(6, 10) // {"0":6,"1":10}
```

下面我们看一下修改参数默认值对arguments的影响。

1、在ES5的非严格模式下，一开始输入的参数是1，那么可以获取到arguments[0]（表示第一个参数）全等于num，修改num = 2之后，arguments[0]也能更新到2。

```javascript
    function a(num){
      console.log(num === arguments[0]) //true
      num = 2 //修改参数默认值
      console.log(num === arguments[0]) //true
    }
    a(1)
```

2、在ES5的严格模式下，arguments就不能在函数内修改默认值后跟随着跟新了。

```javascript
    "use strict"; //严格模式   
    function a(num) {
      console.log(num === arguments[0]); // true
      num = 2;
      console.log(num === arguments[0]); // false
    }
    a(1);
```

**在ES6环境下，默认值对arguments的影响和ES5严格模式是同样的标准。**

**默认参数表达式**

参数不仅可以设置默认值为字符串，数字，数组或者对象，还可以是一个函数。

```javascript
    function add() {
      return 10
    }
    function a(num = add()){
      console.log(num)
    }
    a() // 10
```

**默认参数的临时死区**

第一章我们提到了let和const什么变量的临时死区（TDZ），默认参数既然是参数，那么也同样有临时死区，函数的作用域是独立的，a函数不能共享b函数的作用域参数。

```javascript
    //这是个默认参数临时死区的例子，当初始化a时，b还没有声明，所以第一个参数对b来说就是临时死区。
    function add(a = b, b){
      console.log(a + b)
    }
    add(undefined, 2) // b is not define
```

## 无命名参数

上面说的参数都是命名参数，而无命名参数也是函数传参时经常用到的。当传入的参数是一个对象，不是一个具体的参数名，则是无命名参数。

```javascript
    function add(object){
      console.log(object.a + object.b)
    }
    let obj = {
      a: 1,
      b: 2
    }
    add(obj) // 3
```

**不定参数的使用：**使用...（展开运算符）的参数就是不定参数，它表示一个数组。

```javascript
    function add(...arr){
      console.log(a + b)
    }
    let a = 1,b = 2
    add(a, b) // 3
```

**不定参数的使用限制：**必须放在所有参数的末尾，不能用于对象字面量setter中。

```javascript
    //错误的写法1
    function add(...arr, c){
      console.log(a + b)
    }
    let a = 1,b = 2,c = 3
    add(a, b, c)
    
    //错误的写法2
    let obj = {
      set add(...arr) {
      
      }
    }
```

**ES6中的构造函数Function新增了支持默认参数和不定参数。**

## 展开运算符（...）

展开运算符的作用是解构数组，然后将每个数组元素作为函数参数。

有了展开运算符，我们操作数组的时候，就可以不再使用apply来指定上下文环境了。

```javascript
    //ES5的写法
    let arr = [10, 20, 50, 40, 30]
    let a = Math.max.apply(null, arr)
    console.log(a) // 50
    
    //ES6的写法
    let arr = [10, 20, 50, 40, 30]
    let a = Math.max(...arr)
    console.log(a) // 50
```

## 块级函数

**严格模式下：**在ES6中，你可以在块级作用域内声明函数，该函数的作用域只限于当前块，不能在块的外部访问。

```javascript
    "use strict";
    if(true) {
      const a = function(){
      
      }
    } 
```

**非严格模式：**即使在ES6中，非严格模式下的块级函数，他的作用域也会被提升到父级函数的顶部。所以大家写代码尽量使用严格模式，避免这些奇葩情况。

## 箭头函数（=>）

如果看到你这里，你发现你还没有在项目中使用过箭头函数，没关系，你并不low，而是学习不够努力。

```javascript
    const arr = [5, 10]
    const s = arr.reduce((sum, item) => sum + item)
    console.log(s) // 15
```

**箭头函数和普通函数的区别是：**

1、箭头函数没有this，函数内部的this来自于父级最近的非箭头函数，并且不能改变this的指向。

2、箭头函数没有super

3、箭头函数没有arguments

4、箭头函数没有new.target绑定。

5、不能使用new

6、没有原型

7、不支持重复的命名参数。

**箭头函数的简单理解**

1、箭头函数的左边表示输入的参数，右边表示输出的结果。

```javascript
    const s = a => a
    console.log(s(2)) // 2
```

2、箭头函数中，最重要的this报错将不再成为你每天都担心的bug。

3、箭头函数还可以输出对象，在react的action中就推荐这种写法。

```javascript
    const action = (type, a) => ({
      type: "TYPE",
      a
    })
```

4、支持立即执行函数表达式写法

```javascript
    const test = ((id) => {
      return {
        getId() {
          console.log(id)
        }
      }
    })(18)
    test.getId() // 18
```

5、箭头函数给数组排序

```javascript
    const arr = [10, 50, 30, 40, 20]
    const s = arr.sort((a, b) => a - b)
    console.log(s) // [10,20,30,40,50]
```

## 尾调用优化

尾调用是什么鬼？

尾调用是指在函数return的时候调用一个新的函数，由于尾调用的实现需要存储到内存中，在一个循环体中，如果存在函数的尾调用，你的内存可能爆满或溢出。

ES6中，引擎会帮你做好尾调用的优化工作，你不需要自己优化，但需要满足下面3个要求：

1、函数不是闭包

2、尾调用是函数最后一条语句

3、尾调用结果作为函数返回

**一个满足以上要求的函数如下所示：**

```javascript
    "use strict";   
    function a() {
      return b();
    }
```

**下面的都是不满足的写法：**

```javascript
    //没有return不优化
    "use strict";
    function a() {
      b();
    }
    
    //不是直接返回函数不优化
    "use strict";
    function a() {
      return 1 + b();
    }
    
    //尾调用是函数不是最后一条语句不优化
    "use strict";
    function a() {
      const s = b();
      return s
    }
    
    //闭包不优化
    "use strict";
    function a() {
      const num = 1
      function b() {
        return num
      }
      return b
    }
```

**尾调用实际用途——递归函数优化**

在ES5时代，我们不推荐使用递归，因为递归会影响性能。

但是有了尾调用优化之后，递归函数的性能有了提升。

```javascript
    //新型尾优化写法
    "use strict";  
    function a(n, p = 1) {
      if(n <= 1) {
        return 1 * p
      }
      let s = n * p
      return a(n - 1, s)
    }
    //求 1 x 2 x 3的阶乘
    let sum = a(3)
    console.log(sum) // 6
```

# 四、扩展对象的功能性

## 对象类别

在ES6中，对象分为下面几种叫法。（不需要知道概念）

1、普通对象

2、特异对象

3、标准对象

4、内建对象

## 对象字面量语法拓展

随便打开一个js文件，对象都无处不在，看一个简单的对象。

```javascript
    {
      a: 2
    }
```

**ES6针对对象的语法扩展了一下功能**

1、属性初始值简写

```javascript
    //ES5
    function a(id) {
      return {
        id: id
      };
    };
    
    //ES6
    const a = (id) => ({
      id
    })
```

2、对象方法简写

```javascript
    // ES5
    const obj = {
      id: 1,
      printId: function() {
        console.log(this.id)
      }
    }
    
    // ES6
    const obj = {
      id: 1,
      printId() {
        console.log(this.id)
      }
    }
```

3、属性名可计算

属性名可以传入变量或者常量，而不只是一个固定的字符串。

```javascript
    const id = 5
    const obj = {
      [`my-${id}`]: id
    }
    console.log(obj['my-5']) // 5
```

## ES6对象新增方法

在Object原始对象上新增方法，原则上来说不可取，但是为了解决全世界各地提交的issue，在ES6中的全局Object对象上新增了一些方法。

**1、Object.is()**

用来解决JavaScript中特殊类型 == 或者 === 异常的情况。

下面是一些异常情况

```javascript
    //实际出现了异常(错误输出)
    console.log(NaN === NaN) // false
    console.log(+0 === -0) // true
    console.log(5 == "5") //true
    
    //我们期望的目标输出(正确输出)
    console.log(NaN === NaN) // true
    console.log(+0 === -0) // false
    console.log(5 == "5") //false
```

为了解决历遗留问题，**新增了Object.is()来处理2个值的比较。**

```javascript
    console.log(Object.is(NaN, NaN)) // true
    console.log(Object.is(+0, -0)) // false
    console.log(Object.is(5, "5")) //false
```

**2、Object.assign()**

也许你已经见过或者使用过这个方法了，那这个新增的方法解决了什么问题呢？

答：混合（Mixin）。

mixin是一个方法，实现了拷贝一个对象给另外一个对象，返回一个新的对象。

下面是一个mixin方法的实现，这个方法实现的是浅拷贝。将b对象的属性拷贝到了a对象，合并成一个新的对象。

```javascript
    //mixin不只有这一种实现方法。
    function mixin(receiver, supplier) {
      Object.keys(supplier).forEach((key) => {
        receiver[key] = supplier[key]
      })
      return receiver
    }
    
    let a = {name: 'sb'};
    let b = {
      c: {
        d: 5
        }
      }
    console.log(mixin(a, b)) // {"name":"sb","c":{"d":5}}
```

写这样一个mixin方法是不是很烦，而且每个项目都得引入这个方法，现在，ES6给我们提供了一个现成的方法Object.assign()来做mixin的事情。

假设要实现上面的mixin方法，你只需要给Object.assign()传入参数即可。

```javascript
console.log(Object.assign(a, b))// {"name":"sb","c":{"d":5}}
```

**使用Object.assign()，你就可以不是有继承就能获得另一个对象的所有属性，快捷好用。**
Object.assign 方法只复制源对象中可枚举的属性和对象自身的属性。
**看一个实现Component的例子。**

```javascript
    //声明一个对象Component
    let Component = {}
    //给对象添加原型方法
    Component.prototype = {
      componentWillMount() {},
      componentDidMount() {},
      render() {console.log('render')}
    }
    //定义一个新的对象
    let MyComponent = {}
    //拷贝Component的方法和属性。
    Object.assign(MyComponent, Component.prototype)
    
    console.log(MyComponent.render()) // render
```

**在react的reducer中，每次传入新的参数返回新的state，你都可能用到Object.assign()方法。**

## 重复的对象字面量属性

ES5的严格模式下，如果你的对象中出现了key相同的情况，那么就会抛出错误。而在ES6的严格模式下，不会报错，后面的key会覆盖掉前面相同的key。

```javascript
    const state = {
      id: 1,
      id: 2
    }
    console.log(state.id) // 2
```

## 自有属性枚举顺序

这个概念看起来比较模糊，如果你看了下面的例子，你可能就会明白在说什么了。

```javascript
    const state = {
      id: 1,
      5: 5,
      name: "eryue",
      3: 3
    }
    
    Object.getOwnPropertyNames(state) 
    //["3","5","id","name"] 枚举key

    Object.assign(state, null)
    //{"3":3,"5":5,"id":1,"name":"eryue"} 
```

上面的例子的输出结果都有个规律，就是数字提前，按顺序排序，接着是字母排序。而这种行为也是ES6新增的标准。你还可以自己测试一下其他方法是不是也支持枚举自动排序。比如Object.keys(), for in 等。

## 增强对象原型

如果你想定义一个对象，你会想到很多方法。

```javascript
    let a = {}
    
    let b = Object.create(a)
    
    function C() {}
    
    class D {}
```

那么，ES6是如何在这么强大的对象上面继续增强功能呢？

1、允许改变对象原型

改变对象原型，是指在对象实例化之后，可以改变对象原型。我们使用 `Object.setPrototypeOf()` 来改变实例化后的对象原型。

```javascript
    let a = {
      name() {
        return 'eryue'
      }
    }
    let b = Object.create(a)
    console.log(b.name()) // eryue
      
    //使用setPrototypeOf改变b的原型
    let c = {
      name() {
        return "sb"
      }
    }    
    Object.setPrototypeOf(b, c)    
    console.log(b.name()) //sb
```

2、简化原型访问的super引用

这一个知识你可以看书籍原文，我目前想不到实际业务代码来分析。

## 方法的定义

ES6明确了方法的定义。

```javascript
    let a = {
      //方法
      name() {
        return 'eryue'
      }
    }
    //函数
    function name() {}
```

估计习惯了函数和方法切换的我们，还是不用太在意这些具体的叫法。

# 五、解构

**解构是从对象中提取出更小元素的过程。赋值是对解构出来的元素进行重新赋值。**

## 解构的分类

1、对象解构

2、数组解构

3、混合解构

4、解构参数

## 对象解构

**对象解构简单的例子**

```javascript
    let obj = {
      a: 1,
      b: [1, 2]
    }
    // 对象解构
    const { a, b } = obj
    console.log(a, b) //1  [1, 2]
```

**在函数中使用解构赋值**

解构是将对象或者数组的元素一个个提取出来，而赋值是给元素赋值，解构赋值的作用就是给对象或者数组的元素赋值。

在调用test()函数的时候，我们给参数设置了默认值3，如果不重新赋值，则打印出3,3，但是进行解构赋值后，将props对象的参数解构赋值给a和b，所以打印结果是{a: 1, b: 2}

```javascript
    let props = {
      a: 1,
      b: 2
    }
    function test(value) {
      console.log(value)
    }
    test({a=3, b=3} = props) // {a: 1, b: 2}
```

下面这个例子定义了a = 3,b = 3两个变量，现在我们想修改这2个变量的值，采用解构赋值的方式可以这样做：定义一个props对象，该对象包含2个属性a和b，然后进行解构赋值，这时就能更新变量a和b的value。

```javascript
    let props = {
      a: 1,
      b: 2
    },
    a = 3,
    b = 3;
    //解构赋值
    ({ a, b } = props)
    console.log(a, b) // 1, 2
```

**在react的父子组件传递参数过程中，也使用到了解构赋值。**

```jsx harmony
    class Parent extends React.Component {
      render() {
        const {a = 3, b = 3} = this.props
        return <h1>{a}-{b}</h1>
      }
    }
    
    ReactDOM.render(
      <Parent a="1" b="2" />,
      document.getElementById('root')
    );
    //在浏览器渲染 1-2，默认值是 3-3，但是因为传递了新的props进来，执行了解构赋值之后a和b更新了。
    
```

**嵌套对象解构**

当对象层次较深时，你也可以解构出来。

```javascript
    let obj = {
      a: {
        b: {
          c: 5
        }
      }
    }
    const {a: {b}} = obj
    console.log(b.c) // 5
```

## 数组解构

数组解构比对象解构简单，因为数组只有数组字面量，不需要像对象一个使用key属性。

**数组解构**
你可以选择性的解构元素，不需要解构的元素就使用逗号代替。

```jsx harmony
    let arr = [1, 2, 3]
    
    //解构前2个元素
    const [a, b] = arr
    console.log(a,b) //1 2
    
    //解构中间的元素
    const [, b,] = arr
    console.log(b) // 2
```

**解构赋值**
如果你没有看明白上面说到的对象解构赋值的含义，那么看完下面的数组解构赋值，或许你会有比较清晰的理解。

这个例子中，正常情况下打印a的值是haha，但是将数组arr的第一个元素解构赋值给a，a的值就变成了1。

```javascript
    //初始化一个变量a
    let a = "haha";
    //定义一个数组
    let arr = [1, 2, 3];
    //解构赋值a，将arr数组的第一个元素解构赋值给a，
    [a] = arr;
    console.log(a); // 1
```

使用解构赋值，还可以调换2个变量的值。

```jsx harmony
    let a = 1, b = 2;
    [a, b] = [b, a];
    console.log(a, b); // 2 1 
```

**嵌套数组解构**

```javascript
    let arr = [1, [2, 3], 4];
    let [a, [,b]] = arr;
    console.log(a, b) // 1 3
    
    //实际解构过程，左边的变量和右边的数组元素一一对应下标。
    var a = arr[0],
    _arr$ = arr[1],
    b = _arr$[1];
```

**不定元素解构**
三个点的解构赋值必须放在所有解构元素的最末尾，否则报错。

```javascript
    let arr = [1, 2, 3, 4];
    let [...a] = arr;
    console.log(a) //[1,2,3,4] 这种做法就是克隆arr数组。
```

## 混合解构

混合解构指的是对象和数组混合起来，执行解构操作，没什么难度。

```javascript
    let obj = {
      a: {
        id: 1
      },
      b: [2, 3]
    }
    
    const {
      a: {id},
      b:[...arr]
    } = obj;
    console.log(id, arr) //id = 1, arr = [2, 3]
```

## 解构参数

当给函数传递参数时，我们可以对每个参数进行解构，我给option的参数设置了默认值，这样可以防止没有给option传参导致的报错情况。

```javascript
    function Ajax(url, options) {
      const {timeout = 0, jsonp = true} = options
      console.log(url, timeout, jsonp)
    };
    Ajax('baidu.com', {
      timeout: 1000,
      jsonp: false
    }) // "baidu.com" 1000 false
```

# 六、Symbol和Symbol属性

还记得对象Object吗？

```javascript
    let obj = {
      a: 1
    }
```

对象的格式：

```javascript
    Object {
        key: value
    }
```

在ES5的时代，对象的key只能是字符串String类型。有人就想搞事，把key改成其他数据类型，这不是瞎折腾吗？ES组织的大神们为了对付这类搞事的人，就指定了一个新的数据类型：Symbol。

## 原始数据类型

学习Symbol之前，让我们回忆一下你曾经用过的原始数据类型，只有5个，别搞错了。

**null、undefined**

是不是面试的时候有人问过你这两者的区别？问这种问题的人很无聊，你要是和他当同事，真是受罪。

**Number 数字类型**

```javascript
    const a = 10
    typeof a // number
```

**String 字符串**

```javascript
    const a = 'haha'
    typeof a // string
```

**boolean 布尔型**

```javascript
const a = true, b = false
```

## Symbol

Symbol到底长啥样？又该怎么用呢？我们一起来探索一下。

**在MDN文档中，关于Symbol的说明是这样的：**

Symbol 是一种特殊的、不可变的数据类型，可以作为对象属性的标识符使用。Symbol 对象是一个 symbol primitive data type 的隐式对象包装器。

symbol 数据类型是一个原始数据类型。

**Symbol的语法格式：**

Symbol([description])  //description是可选的

**创建一个Symbol：**

看了Symbol的描述，不知道是什么鬼？长得像个函数。

我们开始按照语法创建一个Symbol来研究一下

```javascript
    const name = Symbol();
    const name1 = Symbol('sym1');
    console.log(name, name1) // Symbol() Symbol(sym1)
```

Symbol不能使用new

```javascript
    const name = new Symbol(); //不可以这样做。
    //Symbol is not a constructor
```

**使用Symbol：**

使用Number的时候，我们可以这样写：

```javascript
    const b = Number(10) // 10
    //简写
    const b = 10
```

同理，使用Symbol，我们可以这样：

```javascript
const name1 = Symbol('sym1'); // Symbol(sym1)
```

在所有使用可计算属性名的地方，都能使用Symbol类型。比如在对象中的key。

```javascript
    const name = Symbol('name');
    const obj = {
      [name]: "haha"
    }
    console.log(obj[name]) // haha
```

你还可以使用Object.defineProperty()和Object.defineProperties()方法。这2个方法是对象的方法，但是作为Symbol类型key，也不影响使用。

```javascript
    // 设置对象属性只读。
    Object.defineProperty(obj, name, {writable: false})
```

这2个方法非常有用，在react源码中，使用了大量的只读属性的对象。以下是从react源码截取的一段代码，设置了props对象只读。但是react仍旧使用字符串作为key，并不用Symbol。

```javascript
    Object.defineProperty(props, 'key', {
        get: warnAboutAccessingKey,
        configurable: true
      });
```

## Symbol全局共享

Symbol有点特殊，在js文件中定义的Symbol，并不能在其他文件直接共享。

ES6提供了一个注册机制，当你注册Symbol之后，就能在全局共享注册表里面的Symbol。Symbol的注册表和对象表很像，都是key、value结构，只不过这个value是Symbol值。
（key, Symbol）
语法：

```javascript
Symbol.for() //只有一个参数
```


还有一个方法是获取注册表的Symbol。

语法：

```javascript
Symbol.keyFor() //只有一个参数，返回的是key
```

从注册表获取全局共享的Symbol

```javascript
    let name = Symbol.for('name');
    let name1 = Symbol.for('name1');
    let name2 = Symbol.for('name2');
    
    console.log(Symbol.keyFor(name)) // name
    console.log(Symbol.keyFor(name1)) // name1
    console.log(Symbol.keyFor(name2)) // name2
```

注意：如果要防止Symbol命名重复问题，可以加上前缀。如：hyy.name

## Symbol与类型强制转换

JavaScript中的类型可以自动转换。比如Number转换成字符串。

```javascript
    let a = 1;
    console.log(typeof a); // number
    console.log(a + ' haha') // '1haha'
```

但是注意了，Symbol不支持这种转换。Symbol就是这么拽啊！

```javascript
    let a = Symbol('a');
    console.log(typeof a);
    console.log(a + ' haha') // Cannot convert a Symbol value to a string
```

## Symbol检索

在对象中获取字符串的key时，可以使用Object.keys()或Object.getOwnPropertyNames()方法获取key，但是使用Symbol做key是，你就只能使用ES6新增的方法来获取了。

```javascript
    let a = Symbol('a');
    let b = Symbol('b');
    
    let obj = {
      [a]: "123",
      [b]: 45
    }
    
    const symbolsKey = Object.getOwnPropertySymbols(obj);
    
    for(let value of symbolsKey) {
      console.log(obj[value]) 
    }
    //"123"
    //45
```

# 七、Set集合与Map集合

Map和Set都叫做集合，但是他们也有所不同。Set常被用来检查对象中是否存在某个键名，Map集合常被用来获取已存的信息。

## Set

### Set是有序列表，含有相互独立的非重复值。

### 创建Set

既然我们现在不知道Set长什么样，有什么价值，那么何不创建一个Set集合看看呢？

**创建一个Set集合，你可以这样做：**

```javascript
    let set = new Set();
    console.log(set);
    
    //在浏览器控制台的输出结果
    Set(0) {}
        size:(...)
        __proto__:Set
        [[Entries]]:Array(0)
        length:0
```

看起来像个对象，那么现在我们在控制台打印一个对象，对比一下两者有什么不同。

```javascript
    let obj = new Object()
    console.log(obj)
    
    //在控制台输出对象
    Object {}
        __proto__: 
```

从输出结果看，Set和Object有明显的区别，反正他们就不是一个东西。

**接着，我们看一下Set的原型有哪些：**

![image-20200607211514258](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Qhg7lONkTEvPWs8.png)

这里主要介绍几个基础原型的作用：

Set.prototype.size  
返回Set对象的值的个数。

Set.prototype.add(value)  
在Set对象尾部添加一个元素。返回该Set对象。

Set.prototype.entries()  
返回一个新的迭代器对象，该对象包含Set对象中的按插入顺序排列的所有元素的值的[value, value]数组。为了使这个方法和Map对象保持相似， 每个值的键和值相等。

Set.prototype.forEach(callbackFn[, thisArg])  
按照插入顺序，为Set对象中的每一个值调用一次callBackFn。如果提供了thisArg参数，回调中的this会是这个参数。

Set.prototype.has(value)  
返回一个布尔值，表示该值在Set中存在与否。

**在例子中使用这几个方法测试一下：**

```javascript
    let set = new Set();
    set.add('haha');
    set.add(Symbol('haha'));
    
    console.log(set.size); //2
    
    console.log(set); 
    Set(2) {"haha", Symbol(haha)}
        size:(...)
        __proto__:Set
        [[Entries]]:Array(2)
            0:"haha"
            1:Symbol(haha)
        length:2
        
    console.log(set.has('haha')) // true
```

到这里，你会发现Set像数组，又像一个对象，但又不完全是。

### 迭代Set

Set既然提供了entries和forEach方法，那么他就是可迭代的。

但如果你使用for in来迭代Set，你不能这样做：

```javascript
    for(let i in sets) {
      console.log(i); //不存在
    }
```

for in迭代的是对象的key，而在Set中的元素没有key，**使用for of来遍历**：

```javascript
    for(let value of sets) {
      console.log(value);
    }
    //"haha"
    //Symbol(haha)
    
    //如果你需要key，则使用下面这种方法
    for(let [key, value] of sets.entries()) {
      console.log(value, key);
    } 
    //"haha" "haha"
    //Symbol(haha) Symbol(haha)
```


**forEach操作Set：**Set本身没有key，而forEach方法中的key被设置成了元素本身。

```javascript
    sets.forEach((value, key) => {
      console.log(value, key);
    });
    //"haha" "haha"
    //Symbol(haha) Symbol(haha)
    
    sets.forEach((value, key) => {
      console.log(Object.is(value, key));
    }); 
    //true true
```

### Set和Array的转换

Set和数组太像了，Set集合的特点是没有key，没有下标，只有size和原型以及一个可迭代的不重复元素的类数组。既然这样，我们就可以把一个Set集合转换成数组，也可以把数组转换成Set。

```javascript
    //数组转换成Set
    const arr = [1, 2, 2, '3', '3']
    let set = new Set(arr);
    console.log(set) // Set(3) {1, 2, "3"}

    //Set转换成数组
    let set = new Set();
    set.add(1);
    set.add('2');
    console.log(Array.from(set)) // (2) [1, "2"]
```

js面试中，经常会考的一道数组去重题目，就可以使用Set集合的不可重复性来处理。经测试只能去重下面3种类型的数据。

```javascript
    const arr = [1, 1, 'haha', 'haha', null, null]
    let set = new Set(arr);
    console.log(Array.from(set)) // [1, 'haha', null]
    console.log([...set]) // [1, 'haha', null]
```

### Weak Set集合

Set集合本身是强引用，只要new Set()实例化的引用存在，就不释放内存，这样一刀切肯定不好啊，比如你定义了一个DOM元素的Set集合，然后在某个js中引用了该实例，但是当页面关闭或者跳转时，你希望该引用应立即释放内存，Set不听话，那好，你还可以使用 [Weak Set][2]

**语法：**

```javascript
new WeakSet([iterable]);
```

**和Set的区别：**

1、**WeakSet 对象中只能存放对象值**, 不能存放原始值, 而 Set 对象都可以.

2、WeakSet 对象中存储的对象值都是被弱引用的, 如果没有其他的变量或属性引用这个对象值, 则这个对象值会被当成垃圾回收掉. 正因为这样, **WeakSet 对象是无法被枚举的**, 没有办法拿到它包含的所有元素.

**使用：**

```javascript
    let set = new WeakSet();
    const class_1 = {}, class_2 = {};
    set.add(class_1);
    set.add(class_2);
    console.log(set) // WeakSet {Object {}, Object {}}
    console.log(set.has(class_1)) // true
    console.log(set.has(class_2)) // true
```

## Map

Map是存储许多键值对的有序列表，key和value支持所有数据类型。

### 创建Map

如果说Set像数组，那么Map更像对象。而对象中的key只支持字符串，Map更加强大，支持所有数据类型，不管是数字、字符串、Symbol等。

```javascript
    // 一个空Map集合
    let map = new Map()
    console.log(map)
```

![image-20200607212020718](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/pxOQVk9im8Lzj4t.png)

**Map的所有原型方法：**
![image-20200607212050017](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/HylFBAU47IMgmP2.png)

对比Set集合的原型，**Map集合的原型多了set()和get()方法**，注意set()和Set集合不是一个东西。Map没有add，使用set()添加key，value，在Set集合中，使用add()添加value，没有key。

```javascript
    let map = new Map();
    map.set('name', 'haha');
    map.set('id', 10);
    console.log(map)
    // 输出结果
    Map(2) {"name" => "haha", "id" => 10}
        size:(...)
        __proto__:Map
        [[Entries]]:Array(2)
            0:{"name" => "haha"}
            1:{"id" => 10}
        length:2

    console.log(map.get('id')) // 10
    console.log(map.get('name')) // "haha"
```

**使用对象做key**

```javascript
    let map = new Map();
    const key = {};
    map.set(key, '谁知道这是个什么玩意');
    console.log(map.get(key)) // 谁知道这是个什么玩意
```

**Map同样可以使用forEach遍历key、value**

```javascript
    let map = new Map();
    const key = {};
    map.set(key, '这是个什么玩意');
    map.set('name', 'haha');
    map.set('id', 1);
    map.forEach((value, key) => {
      console.log(key, value)
    })
    
    //Object {} "这是个什么玩意"
    //"name" "haha"
    //"id" 1
```

### Weak Map

有强Map，就有弱鸡Map。

和Set要解决的问题一样，希望不再引用Map的时候自动触发垃圾回收机制。那么，你就需要Weak Map。

```javascript
    let map = new WeakMap();
    const key = document.querySelector('.header');
    map.set(key, '这是个什么玩意');
    
    map.get(key) // "这是个什么玩意"
    
    //移除该元素
    key.parentNode.removeChild(key);
    key = null;
```

>Set集合可以用来过滤数组中重复的元素，只能通过has方法检测指定的值是否存在，或者是通过forEach处理每个值。
>
>Weak Set集合存放对象的弱引用，当该对象的其他强引用被清除时，集合中的弱引用也会自动被垃圾回收机制回收，追踪成组的对象是该集合最好的使用方式。
>
>Map集合通过set()添加键值对，通过get()获取键值，各种方法的使用查看文章教程，你可以把它看成是比Object更加强大的对象。
>
>Weak Map集合只支持对象类型的key，所有key都是弱引用，当该对象的其他强引用被清除时，集合中的弱引用也会自动被垃圾回收机制回收，为那些实际使用与生命周期管理分离的对象添加额外信息是非常适合的使用方式。

# 八、迭代器（Iterator）

## ES5实现迭代器

**迭代器是一种特殊对象，每一个迭代器对象都有一个next()，该方法返回一个对象，包括value和done属性。**

**ES5实现迭代器的代码如下：**

```javascript
    //实现一个返回迭代器对象的函数，注意该函数不是迭代器，返回结果才叫做迭代器。
    function createIterator(items) {
      var i = 0;
      return {
        next() {
          var done = (i >= items.length); // 判断i是否小于遍历的对象长度。
          var value = !done ? items[i++] : undefined; //如果done为false，设置value为当前遍历的值。
          return {
            done,
            value
          }
        }
      }
    }
    const a = createIterator([1, 2, 3]);
    
    //该方法返回的最终是一个对象，包含value、done属性。
    console.log(a.next()); //{value: 1, done: false}
    console.log(a.next()); //{value: 2, done: false}
    console.log(a.next()); //{value: 3, done: false}
    console.log(a.next()); //{value: undefined, done: true}
```

## 生成器（Generator）

**生成器是函数：用来返回迭代器。**

这个概念有2个关键点，一个是函数、一个是返回迭代器。这个函数不是上面ES5中创建迭代器的函数，而是ES6中特有的，一个带有*（星号）的函数，同时你也需要使用到yield。

```javascript
    //生成器函数，ES6内部实现了迭代器功能，你要做的只是使用yield来迭代输出。
    function *createIterator() {
      yield 1;
      yield 2;
      yield 3;
    }
    const a = createIterator();
    console.log(a.next()); //{value: 1, done: false}
    console.log(a.next()); //{value: 2, done: false}
    console.log(a.next()); //{value: 3, done: false}
    console.log(a.next()); //{value: undefined, done: true}
```

生成器的yield关键字有个神奇的功能，就是当你执行一次next()，**那么只会执行一个yield后面的内容，然后语句终止运行**。

## 在for循环中使用迭代器

即使你是在for循环中使用yield关键字，也会暂停循环。

```javascript
    function *createIterator(items) {
      for(let i = 0; i < items.length;  i++) {
        yield items[i]
      }
    }
    const a = createIterator([1, 2, 3]);
    console.log(a.next()); //{value: 1, done: false}
```

## yield使用限制

yield只可以在生成器函数内部使用，如果在非生成器函数内部使用，则会报错。

```javascript
    function *createIterator(items) {
        //你应该在这里使用yield
      items.map((value, key) => {
        yield value //语法错误，在map的回调函数里面使用了yield
      })
    }
    const a = createIterator([1, 2, 3]);
    console.log(a.next()); //无输出
```

## 生成器函数表达式

函数表达式很简单，就是下面这种写法，也叫匿名函数，不用纠结。

```javascript
    const createIterator = function *() {
        yield 1;
        yield 2;
    }
    const a = createIterator();
    console.log(a.next());
```

## 在对象中添加生成器函数

一个对象长这样：

```javascript
const obj = {}
```

我们可以在obj中添加一个生成器，也就是添加一个带星号的方法：

```javascript
    const obj = {
      a: 1,
      *createIterator() {
        yield this.a
      }
    }
    const a = obj.createIterator();
    console.log(a.next());  //{value: 1, done: false}
```

## 可迭代对象和for of循环

再次默读一遍，迭代器是对象，生成器是返回迭代器的函数。

凡是通过生成器生成的迭代器，都是可以迭代的对象(可迭代对象具有Symbol.iterator属性)，也就是可以通过for of将value遍历出来。

```javascript
    function *createIterator() {
      yield 1;
      yield 2;
      yield 3;
    }
    const a = createIterator();
    for(let value of a) {
      console.log(value)
    }
    // 1 2 3
```

上面的例子告诉我们生成器函数返回的迭代器是一个可以迭代的对象。其实我们这里要研究的是Symbol.iterator的用法。

```javascript
    function *createIterator() {
      yield 1;
      yield 2;
      yield 3;
    }
    const a = createIterator(); //a是一个迭代器
    const s = a[Symbol.iterator]();//使用Symbol.iterator访问迭代器
    console.log(s.next()) //{value: 1, done: false}
```

Symbol.iterator还可以用来检测一个对象是否可迭代：

```javascript
typeof obj[Symbol.iterator] === "function"
```

## 创建可迭代对象

**在ES6中，数组、Set、Map、字符串都是可迭代对象。**

**默认情况下定义的对象（object）是不可迭代的，但是可以通过Symbol.iterator创建迭代器。**

```javascript
    const obj = {
      items: []
    }
    obj.items.push(1);//这样子虽然向数组添加了新元素，但是obj不可迭代
    for (let x of obj) {
      console.log(x) // _iterator[Symbol.iterator] is not a function
    }

    //接下来给obj添加一个生成器，使obj成为一个可以迭代的对象。
    const obj = {
      items: [],
      *[Symbol.iterator]() {
        for (let item of this.items) {
          yield item;
        }
      }
    }
    obj.items.push(1)
    //现在可以通过for of迭代obj了。
    for (let x of obj) {
      console.log(x)
    }
```

## 内建迭代器

上面提到了，数组、Set、Map都是可迭代对象，即它们内部实现了迭代器，并且提供了3种迭代器函数调用。

**1、entries() 返回迭代器**：返回键值对

```javascript
    //数组
    const arr = ['a', 'b', 'c'];
    for(let v of arr.entries()) {
      console.log(v)
    }
    // [0, 'a'] [1, 'b'] [2, 'c']
    
    //Set
    const arr = new Set(['a', 'b', 'c']);
    for(let v of arr.entries()) {
      console.log(v)
    }
    // ['a', 'a'] ['b', 'b'] ['c', 'c']

    //Map
    const arr = new Map();
    arr.set('a', 'a');
    arr.set('b', 'b');
    for(let v of arr.entries()) {
      console.log(v)
    }
    // ['a', 'a'] ['b', 'b']
```


**2、values() 返回迭代器**：返回键值对的value

```javascript
    //数组
    const arr = ['a', 'b', 'c'];
    for(let v of arr.values()) {
      console.log(v)
    }
    //'a' 'b' 'c'

    //Set
    const arr = new Set(['a', 'b', 'c']);
    for(let v of arr.values()) {
      console.log(v)
    }
    // 'a' 'b' 'c'

    //Map
    const arr = new Map();
    arr.set('a', 'a');
    arr.set('b', 'b');
    for(let v of arr.values()) {
      console.log(v)
    }
    // 'a' 'b'
```

**3、keys() 返回迭代器**：返回键值对的key

```javascript
    //数组
    const arr = ['a', 'b', 'c'];
    for(let v of arr.keys()) {
      console.log(v)
    }
    // 0 1 2
    
    //Set
    const arr = new Set(['a', 'b', 'c']);
    for(let v of arr.keys()) {
      console.log(v)
    }
    // 'a' 'b' 'c'

    //Map
    const arr = new Map();
    arr.set('a', 'a');
    arr.set('b', 'b');
    for(let v of arr.keys()) {
      console.log(v)
    }
    // 'a' 'b'
```

虽然上面列举了3种内建的迭代器方法，但是不同集合的类型还有自己默认的迭代器，在for of中，数组和Set的默认迭代器是values()，Map的默认迭代器是entries()。
​    

## for of循环解构

对象本身不支持迭代，但是我们可以自己添加一个生成器，返回一个key，value的迭代器，然后使用for of循环解构key和value。

```javascript
    const obj = {
      a: 1,
      b: 2,
      *[Symbol.iterator]() {
        for(let i in obj) {
          yield [i, obj[i]]
        }
      }
    }
    for(let [key, value] of obj) {
      console.log(key, value)
    }
    // 'a' 1, 'b' 2
```

## 字符串迭代器

```javascript
    const str = 'abc';
    for(let v of str) {
      console.log(v)
    }
    // 'a' 'b' 'c'
```

## NodeList迭代器

迭代器真是无处不在啊，dom节点的迭代器你应该已经用过了。

```javascript
    const divs = document.getElementByTagName('div');
    for(let d of divs) {
      console.log(d)
    }
```

## 展开运算符和迭代器

```javascript
    const a = [1, 2, 3];
    const b = [4, 5, 6];
    const c = [...a, ...b]
    console.log(c) // [1, 2, 3, 4, 5, 6]
```

## 高级迭代器功能

高级功能不复杂，就是传参、抛出异常、生成器返回语句、委托生成器。

1、传参

生成器里面有2个yield，当执行第一个next()的时候，返回value为1，然后给第二个next()传入参数10，传递的参数会替代掉上一个next()的yield返回值。在下面的例子中就是first。

```javascript
    function *createIterator() {
      let first = yield 1;
      yield first + 2;
    }
    let i = createIterator();
    console.log(i.next()); // {value: 1, done: false}
    console.log(i.next(10)); // {value: 12, done: false}
```

2、在迭代器中抛出错误

```javascript
    function *createIterator() {
      let first = yield 1;
      yield first + 2;
    }
    let i = createIterator();
    console.log(i.next()); // {value: 1, done: false}
    console.log(i.throw(new Error('error'))); // error
    console.log(i.next()); //不再执行
```

3、生成器返回语句

生成器中添加return表示退出操作。

```javascript
    function *createIterator() {
      let first = yield 1;
      return;
      yield first + 2;
    }
    let i = createIterator();
    console.log(i.next()); // {value: 1, done: false}
    console.log(i.next()); // {value: undefined, done: true}
```

4、委托生成器

生成器嵌套生成器

```javascript
    function *aIterator() {
      yield 1;
    }
    function *bIterator() {
      yield 2;
    }
    function *cIterator() {
      yield *aIterator()
      yield *bIterator()
    }
    
    let i = cIterator();
    console.log(i.next()); // {value: 1, done: false}
    console.log(i.next()); // {value: 2, done: false}
```

## 异步任务执行器

ES6之前，我们使用异步的操作方式是调用函数并执行回调函数。

书上举的例子挺好的，在nodejs中，有一个读取文件的操作，使用的就是回调函数的方式。

```javascript
    var fs = require("fs");
    fs.readFile("xx.json", function(err, contents) {
      //在回调函数中做一些事情
    })
```

那么任务执行器是什么呢？

**任务执行器是一个函数，用来循环执行生成器，因为我们知道生成器需要执行N次next()方法，才能运行完，所以我们需要一个自动任务执行器帮我们做这些事情，这就是任务执行器的作用。**    

下面我们编写一个异步任务执行器。

```javascript
    //taskDef是一个生成器函数，run是异步任务执行器
    function run(taskDef) {
      let task = taskDef(); //调用生成器
      let result = task.next(); //执行生成器的第一个next()，返回result
      function step() {
        if(!result.done) {
        //如果done为false，则继续执行next()，并且循环step，直到done为true退出。
          result = task.next(result.value);
          step();
        }
      }
      step(); //开始执行step()
    }
```

测试一下我们编写的run方法，我们不再需要console.log N个next了，因为run执行器已经帮我们做了循环执行操作：

```javascript
    run(function *() {
      let value = yield 1;
      value = yield value + 20;
      console.log(value) // 21
    })
```

# 九、类class

## ES5中的近类结构

ES5以及之前的版本，没有类的概念，但是聪明的JavaScript开发者，为了实现面向对象，创建了特殊的近类结构。

**ES5中创建类的方法：新建一个构造函数，定义一个方法并且赋值给构造函数的原型。**

```javascript
    'use strict';
    //新建构造函数，默认大写字母开头
    function Person(name) {
      this.name = name;
    }
    //定义一个方法并且赋值给构造函数的原型
    Person.prototype.sayName = function () {
      return this.name;
    };
    
    var p = new Person('eryue');
    console.log(p.sayName() // eryue
    );
```

## ES6 class类

如果你学过java，那么一定会非常熟悉这种声明类的方式。

```javascript
    class Person {
      //新建构造函数
      constructor(name) {
        this.name = name //私有属性
      }
      //定义一个方法并且赋值给构造函数的原型
      sayName() {
        return this.name
      }
    }
    let p = new Person('eryue')
    console.log(p.sayName()) // eryue
```

和ES5中使用构造函数不同的是，在ES6中，我们将原型的实现写在了类中，但本质上还是一样的，都是需要新建一个类名，然后实现构造函数，再实现原型方法。

**私有属性：**在class中实现私有属性，只需要在构造方法中定义this.xx = xx。

## 类声明和函数声明的区别和特点

1、函数声明可以被提升，类声明不能提升。

2、类声明中的代码自动强行运行在严格模式下。

3、类中的所有方法都是不可枚举的，而自定义类型中，可以通过Object.defineProperty()手工指定不可枚举属性。

4、每个类都有一个**construct**的方法。

5、只能使用new来调用类的构造函数。

6、不能在类中修改类名。

## 类表达式

类有2种表现形式：声明式和表达式。

```javascript
    //声明式
    class B {
      constructor() {}
    }

    //匿名表达式
    let A = class {
      constructor() {}
    }
    
    //命名表达式，B可以在外部使用，而B1只能在内部使用
    let B = class B1 {
      constructor() {}
    }
```

## 类是一等公民

JavaScript函数是一等公民，类也设计成一等公民。

1、可以将类作为参数传入函数。

```javascript
    //新建一个类
    let A = class {
      sayName() {
        return 'eryue'
      }
    }
    //该函数返回一个类的实例
    function test(classA) {
      return new classA()
    }
    //给test函数传入A
    let t = test(A)
    console.log(t.sayName()) // eryue
```

2、通过立即调用类构造函数可以创建单例。

```javascript
    let a = new class {
      constructor(name) {
        this.name = name
      }
      sayName() {
        return this.name
      }
    }('eryue')
    console.log(a.sayName()) // eryue
```

## 访问器属性

类支持在原型上定义访问器属性。

```javascript
    class A {
      constructor(state) {
        this.state = state
      }
      // 创建getter
      get myName() {
        return this.state.name
      }
      // 创建setter
      set myName(name) {
        this.state.name = name
      }
    }
    // 获取指定对象的自身属性描述符。自身属性描述符是指直接在对象上定义（而非从对象的原型继承）的描述符。
    let desriptor = Object.getOwnPropertyDescriptor(A.prototype, "myName")
    console.log("get" in desriptor) // true
    console.log(desriptor.enumerable) // false 不可枚举
```

## 可计算成员名称

可计算成员是指使用方括号包裹一个表达式，如下面定义了一个变量m，然后使用[m]设置为类A的原型方法。

```javascript
    let m = "sayName"
    class A {
      constructor(name) {
        this.name = name
      }
      [m]() {
        return this.name
      }
    }
    let a = new A("eryue")
    console.log(a.sayName()) // eryue
```

## 生成器方法

回顾一下上一章讲的生成器，生成器是一个返回迭代器的函数。在类中，我们也可以使用生成器方法。

```javascript
    class A {
      *printId() {
        yield 1;
        yield 2;
        yield 3;
      }
    }
    let a = new A()
    let x = a.printId()
    
    x.next() // {done: false, value: 1}
    x.next() // {done: false, value: 2}
    x.next() // {done: false, value: 3}
```

这个写法很有趣，我们新增一个原型方法稍微改动一下。

```javascript
    class A {
      *printId() {
        yield 1;
        yield 2;
        yield 3;
      }
      render() {
      //从render方法访问printId，很熟悉吧，这就是react中经常用到的写法。
        return this.printId()
      }
    }
    let a = new A()
    console.log(a.render().next()) // {done: false, value: 1}
```

## 静态成员

静态成员是指在方法名或属性名前面加上static关键字，和普通方法不一样的是，static修饰的方法不能在实例中访问，只能在类中直接访问。

```javascript
    class A {
      constructor(name) {
        this.name = name
      }
      static create(name) {
        return new A(name)
      }
    }
    let a = A.create("eryue")
    console.log(a.name) // eryue
    
    let t = new A()
    console.log(t.create("eryue")) // t.create is not a function
```

## 继承与派生类

我们在写react的时候，自定义的组件会继承React.Component。

```javascript
    class A extends Component {
       constructor(props){
           super(props)
       }
    }
```

**A叫做派生类**，在派生类中，如果使用了构造方法，就必须使用super()。

```javascript
    class Component {
      constructor([a, b] = props) {
        this.a = a
        this.b = b
      }
      add() {
        return this.a + this.b
      }
    }
    
    class T extends Component {
      constructor(props) {
        super(props)
      }
    }
    
    let t = new T([2, 3])
    console.log(t.add()) // 5
```

**关于super使用的几点要求：**

1、只可以在派生类中使用super。派生类是指继承自其它类的新类。

2、在构造函数中访问this之前要调用super()，负责初始化this。

```javascript
    class T extends Component {
      constructor(props) {
        this.name = 1 // 错误，必须先写super()
        super(props)
      }
    }
```

3、如果不想调用super，可以让类的构造函数返回一个对象。

## 类方法遮蔽

我们可以在继承的类中重写父类的方法。

```javascript
    class Component {
      constructor([a, b] = props) {
        this.a = a
        this.b = b
      }
      //父类的add方法，求和
      add() {
        return this.a + this.b
      }
    }
    
    class T extends Component {
      constructor(props) {
        super(props)
      }
      //重写add方法，求积
      add() {
        return this.a * this.b
      }
    }
    let t = new T([2, 3])
    console.log(t.add()) // 6
```

## 静态成员继承

**父类中的静态成员，也可以继承到派生类中。**静态成员继承只能通过派生类访问，不能通过派生类的实例访问。

```javascript
    class Component {
      constructor([a, b] = props) {
        this.a = a
        this.b = b
      }
      static printSum([a, b] = props) {
        return a + b
      }
    }
    class T extends Component {
      constructor(props) {
        super(props)
      }
    }
    console.log(T.printSum([2, 3])) // 5
```

## 派生自表达式的类

很好理解，就是指父类可以是一个表达式。

## 内建对象的继承

有些牛逼的人觉得使用内建的Array不够爽，就希望ECMA提供一种继承内建对象的方法，然后那帮大神们就把这个功能添加到class中了。

```javascript
    class MyArray extends Array { }
    let colors = new MyArray()
    colors[0] = "1"
    console.log(colors) // [1]
```

## 在构造函数中使用new.target

new.target通常表示当前的构造函数名。通常我们使用new.target来阻止直接实例化基类，下面是这个例子的实现。

```javascript
    class A {
      constructor() {
      //如果当前的new.target为A类，就抛出异常
        if (new.target === A) {
          throw new Error("error haha")
        }
      }
    }
    let a = new A()
    console.log(a) // error haha
```

# 十、数组

ES5提供的数组已经很强大，但是ES6中继续改进了一些，主要是增加了新的数组方法，所以这章的知识非常少。

## 创建数组

**ES5中创建数组的方式：数组字面量、new一个数组。**

```javascript
    const arr1 = [] //数组字面量
    const arr2 = new Array() //new构建
```

**ES6创建数组：Array.of()、Array.from()**

### Array.of()

ES5中new一个人数组的时候，会存在一个令人困惑的情况。当new一个数字的时候，生成的是一个长度为该数字的数组，当new一个字符串的时候，生成的是该字符串为元素的数组。

```javascript
    const a = new Array(2)
    const b = new Array("2")
    console.log(a, b) //[undefined, undefined] ["2"]
```

这样一来，导致new Array的行为是不可预测的，Array.of()出现为的就是解决这个情况。

```javascript
    const c = Array.of(2)
    const d = Array.of("2")
    console.log(c, d) // [2] ["2"]
```

使用Array.of()创建的数组传入的参数都是作为数组的元素，而不在是数组长度，这样就避免了使用上的歧义。

### Array.from()

如果说Array.of()是创建一个新数组，而**Array.from()是将类数组转换成数组**。

下面的例子讲的是将arguments转换成数组。arguments是类数组对象，他表示的是当前函数的所有参数，如果函数没有参数，那么arguments就为空。

```javascript
    function test(a, b) {
      let arr = Array.from(arguments)
      console.log(arr)
    }
    test(1, 2) //[1, 2]
```

**映射转换：**Array.from(arg1, arg2)，我们可以给该方法提供2个参数，第二个参数作为第一个参数的转换。看个简单例子你就懂了。

```javascript
    function test(a, b) {
      let arr = Array.from(arguments, value => value + 2)
      console.log(arr)
    }
    test(1, 2) //[3, 4]
```

Array.from还可以设置第三个参数，指定this。

**Array.from()转换可迭代对象：**这个用法只需要一个例子，数组去重。

```javascript
    function test() {
      return Array.from(new Set(...arguments))
    }
    const s = test([1, "2", 3, 3, "2"])
    console.log(s) // [1,"2",3]
```

## 新方法

**ES6给数组添加了几个新方法：find()、findIndex()、fill()、copyWithin()。**

1、find()：传入一个回调函数，找到数组中符合当前搜索规则的第一个元素，返回它，并且终止搜索。

```javascript
    const arr = [1, "2", 3, 3, "2"]
    console.log(arr.find(n => typeof n === "number")) // 1
```

2、findIndex()：传入一个回调函数，找到数组中符合当前搜索规则的第一个元素，返回它的下标，终止搜索。

```javascript
    const arr = [1, "2", 3, 3, "2"]
    console.log(arr.findIndex(n => typeof n === "number")) // 0
```

3、fill()：用新元素替换掉数组内的元素，可以指定替换下标范围。

```javascript
    arr.fill(value, start, end)
```

测试一下

```javascript
    const arr = [1, 2, 3]
    console.log(arr.fill(4)) // [4, 4, 4] 不指定开始和结束，全部替换
    
    const arr1 = [1, 2, 3]
    console.log(arr1.fill(4, 1)) // [1, 4, 4] 指定开始位置，从开始位置全部替换
    
    const arr2 = [1, 2, 3]
    console.log(arr2.fill(4, 0, 2)) // [4, 4, 3] 指定开始和结束位置，替换当前范围的元素
```

4、copyWithin()：选择数组的某个下标，从该位置开始复制数组元素，默认从0开始复制。也可以指定要复制的元素范围。

```javascript
    arr.copyWithin(target, start, end)
```

测试一下

```javascript
    const arr = [1, 2, 3, 4, 5]
    console.log(arr.copyWithin(3)) // [1,2,3,1,2] 从下标为3的元素开始，复制数组，所以4, 5被替换成1, 2
    
    const arr1 = [1, 2, 3, 4, 5]
    console.log(arr1.copyWithin(3, 1)) // [1,2,3,2,3] 从下标为3的元素开始，复制数组，指定复制的第一个元素下标为1，所以4, 5被替换成2, 3
    
    const arr2 = [1, 2, 3, 4, 5]
    console.log(arr2.copyWithin(3, 1, 2)) // [1,2,3,2,5] 从下标为3的元素开始，复制数组，指定复制的第一个元素下标为1，结束位置为2，所以4被替换成2
```

# 十一、Promise和异步编程

## 为什么要异步编程

我们在写前端代码时，经常会对dom做事件处理操作，比如点击、激活焦点、失去焦点等；再比如我们用ajax请求数据，使用回调函数获取返回值。这些都属于异步编程。

也许你已经大概知道JavaScript引擎单线程的概念，那么这种单线程模式和异步编程有什么关系呢？

**JavaScript引擎中，只有一个主线程，当执行JavaScript代码块时，不允许其他代码块执行，而事件机制和回调机制的代码块会被添加到任务队列中，当符合某个触发回调或者事件的时候，就会执行该事件或者回调函数。**

**事件模型：**
浏览器初次渲染DOM的时候，我们会给一些DOM绑定事件函数，只有当触发了这些DOM事件函数，才会执行他们。

```javascript
    const btn = document.querySelector('.button')
    btn.onclick = function(event) {
      console.log(event)
    }
```

**回调模式：**
nodejs中可能非常常见这种回调模式，但是对于前端来说，ajax的回调是最熟悉不过了。ajax回调有多个状态，当响应成功和失败都有不同的回调函数。

```javascript
    $.post('/router', function(data) {
      console.log(data)
    })
```

## Promise

事件函数没有问题，我们用的很爽，问题出在回调函数，尤其是指地狱回调，Promise的出现正是为了避免地狱回调带来的困扰。

### Promise是什么

Promise的中文意思是承诺，也就是说，JavaScript对你许下一个承诺，会在未来某个时刻兑现承诺。

### Promise生命周期

react有生命周期，vue也有生命周期，就连Promise也有生命周期。

**Promise的生命周期：进行中（pending），已经完成（fulfilled），拒绝（rejected）**

Promise被称作异步结果的占位符，它不能直接返回异步函数的执行结果，需要使用then()，当获取异常回调的时候，使用catch()。

这次我们使用axios插件的代码做例子。axios是前端比较热门的http请求插件之一。

1、创建axios实例instance。

```javascript
    import axios from 'axios'
    export const instance = axios.create()
```

2、使用axios实例 + Promise获取返回值。

```javascript
    const promise = instance.get('url')
    
    promise.then(result => console.log(result)).catch(err => console.log(err))
```

### 使用Promise构建函数创建新的Promise

Promise构造函数只有一个参数，该参数是一个函数，被称作执行器，执行器有2个参数，分别是resolve()和reject()，一个表示成功的回调，一个表示失败的回调。

```javascript
    new Promise(function(resolve, reject) {
      setTimeout(() => resolve(5), 0)
    }).then(v => console.log(v)) // 5
```

**记住，Promise实例只能通过resolve或者reject函数来返回，并且使用then()或者catch()获取，不能在new Promise里面直接return，这样是获取不到Promise返回值的。**

1、我们也可以使用Promise直接resolve(value)。

```javascript
    Promise.resolve(5).then(v => console.log(v)) // 5
```

2、也可以使用reject(value)

```javascript
    Promise.reject(5).catch(v => console.log(v)) // 5
```

3、执行器错误通过catch捕捉。

```javascript
    new Promise(function(resolve, reject) {
        if(true) {
           throw new Error('error!!')
        }
    }).catch(v => console.log(v.message)) // error!!
```

### 全局的Promise拒绝处理

不重要的内容，不用细看。

这里涉及到nodejs环境和浏览器环境的全局，主要说的是如果执行了Promise.reject()，浏览器或者node环境并不会强制报错，只有在你调用catch的时候，才能知道Promise被拒绝了。

这种行为就像是，你写了一个函数，函数内部有true和false两种状态，而我们希望false的时候抛出错误，但是在Promise中，并不能直接抛出错误，**无论Promise是成功还是拒绝状态，你获取Promise生命周期的方法只能通过then()和catch()。**

**nodejs环境：**

node环境下有个对象叫做process，即使你没写过后端node，如果写过前端node服务器，也应该知道可以使用process.ENV_NODE获取环境变量。为了监听Promise的reject（拒绝）情况，NodeJS提供了一个process.on()，类似jQuery的on方法，事件绑定函数。

process.on()有2个事件

unhandledRjection:在一个事件循环中，当Promise执行reject()，并且没有提供catch()时被调用。

正常情况下，你可以使用catch捕捉reject。

 ```javascript
    Promise.reject("It was my wrong!").catch(v => console.log(v))
 ```

但是，有时候你不总是记得使用catch。你就需要使用process.on()

```javascript
    let rejected
    rejected = Promise.reject("It was my wrong!")
    
    process.on("unhandledRjection", function(reason, promise) {
      console.log(reason.message) // It was my wrong!
      console.log(rejected === promise) // true
    })
```


rejectionHandled:在一个事件循环后，当Promise执行reject，并且没有提供catch()时被调用。

```javascript
    let rejected
    rejected = Promise.reject(new Error("It was my wrong!"))
    
    process.on("rejectionHandled", function(promise) {
      console.log(rejected === promise) // true
    })
```

**异同：**

事件循环中、事件循环后，你可能很难理解这2个的区别，但是这不重要，重要的是，如果你通过了catch()方法来捕捉reject操作，那么，这2个事件就不会生效。

**浏览器环境：**

和node环境一样，都提供了unhandledRjection、rejectionHandled事件，不同的是浏览器环境是通过window对象来定义事件函数。

```javascript
    let rejected
    rejected = Promise.reject(new Error("It was my wrong!"))
    
    window.rejectionHandled = function(event) {
      console.log(event) // true
    }
    rejectionHandled()
```

将代码在浏览器控制台执行一遍，你就会发现报错了：Uncaught (in promise) Error: It was my wrong!

耶，你成功了！报错内容正是你写的reject()方法里面的错误提示。

### Promise链式调用

这个例子中，使用了3个then，第一个then返回 s * s，第二个then捕获到上一个then的返回值，最后一个then直接输出end。这就叫链式调用，很好理解的。我只使用了then()，实际开发中，你还应该加上catch()。

```javascript
    new Promise(function(resolve, reject) {
      try {
        resolve(5)
      } catch (error) {
        reject('It was my wrong!!!')
      }
    }).then(s => s * s).then(s2 => console.log(s2)).then(() => console.log('end'))
    // 25  "end"
```

### Promise的其他方法

在Promise的构造函数中，除了reject()和resolve()之外，还有2个方法，Promise.all()、Promise.race()。

**Promise.all()：**

前面我们的例子都是只有一个Promise，现在我们使用all()方法包装多个Promise实例。

语法很简单：参数只有一个，可迭代对象，可以是数组，或者Symbol类型等。

```javascript
    Promise.all(iterable).then().catch()
```

示例：传入3个Promise实例

```javascript
    Promise.all([
      new Promise(function(resolve, reject) {
        resolve(1)
      }),
      new Promise(function(resolve, reject) {
        resolve(2)
      }),
      new Promise(function(resolve, reject) {
        resolve(3)
      })
    ]).then(arr => {
      console.log(arr) // [1, 2, 3]
    })
```

**Promise.race()：**语法和all()一样，但是返回值有所不同，race根据传入的多个Promise实例，只要有一个实例resolve或者reject，就只返回该结果，其他实例不再执行。

还是使用上面的例子，只是我给每个resolve加了一个定时器，最终结果返回的是3，因为第三个Promise最快执行。

```javascript
    Promise.race([
      new Promise(function(resolve, reject) {
        setTimeout(() => resolve(1), 1000)
      }),
      new Promise(function(resolve, reject) {
        setTimeout(() => resolve(2), 100)
      }),
      new Promise(function(resolve, reject) {
        setTimeout(() => resolve(3), 10)
      })
    ]).then(value => {
      console.log(value) // 3
    })
```

### Promise派生

派生的意思是定义一个新的Promise对象，继承Promise方法和属性。

```javascript
    class MyPromise extends Promise {
    
      //重新封装then()
      success(resolve, reject) {
        return this.then(resolve, reject)
      }
      //重新封装catch()
      failer(reject) {
        return this.catch(reject)
      }
    }
```

接着我们来使用一下这个派生类。
​    

```javascript
    new MyPromise(function(resolve, reject) {
      resolve(10)
    }).success(v => console.log(v)) // 10
```

如果只是派生出来和then、catch一样的方法，我想，你不会干这么无聊的事情。

### Promise和异步的联系

Promise本身不是异步的，只有他的then()或者catch()方法才是异步，也可以说Promise的返回值是异步的。通常Promise被使用在node，或者是前端的ajax请求、前端DOM渲染顺序等地方。

# 十二、代理和反射

## 反射 Reflect

当你见到一个新的API，不明白的时候，就在浏览器打印出来看看它的样子。

![image-20200608102122133](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/4fGzcvKXAgenZMC.png)

### 反射的概念

Reflect 是一个内置的对象，它提供可拦截JavaScript操作的方法。方法与代理处理程序的方法相同。Reflect 不是一个函数对象，因此它是不可构造的。

```javascript
    new Reflect() //错误的写法
```

### 反射的使用

**静态方法列表：**这么多静态方法，**你需要学会的是如何使用它们**。

1、Reflect.apply()
2、Reflect.construct()
3、Reflect.defineProperty()
4、Reflect.deleteProperty()
5、Reflect.enumerate()
6、Reflect.get()
7、Reflect.getOwnPropertyDescriptor()
8、Reflect.getPrototypeOf()
9、Reflect.has()
10、Reflect.isExtensible()
11、Reflect.ownKeys()
12、Reflect.preventExtensions()
13、Reflect.set()
14、Reflect.setPrototypeOf()

**静态方法的使用：**

demo1：使用Reflect.get()获取目标对象指定key的value。

```javascript
    let obj = {
        a: 1
    };
    
    let s1 = Reflect.get(obj, "a")
    console.log(s1) // 1
```

demo2:使用Reflect.apply给目标函数floor传入指定的参数。

```javascript
    const s2 = Reflect.apply(Math.floor, undefined, [1.75]); 
    console.log(s2) // 1
```

### 进一步理解Reflect

看了上面的例子和方法，我们知道Reflect可以拦截JavaScript代码，包括拦截对象，拦截函数等，然后对拦截到的对象或者函数进行读写等操作。

比如demo1的get()方法，拦截obj对象，然后读取key为a的值。当然，不用反射也可以读取a的值。

再看demo2的apply()方法，这个方法你应该比较了解了，和数组中使用apply不同的是，Reflect.apply()提供了3个参数，第一个参数是反射的函数，后面2个参数才是和数组的apply一致。demo2的例子我们可以理解成是拦截了Math.floor方法，然后传入参数，将返回值赋值给s2，这样我们就能在需要读取这个返回值的时候调用s2。

```javascript
    //数组使用apply
    const arr = [1, 2, 3]
    function a() {
      return Array.concat.apply(null, arguments)
    }
    const s = a(arr)
    console.log(s) // [1, 2 ,3]
```

其实Reflect的作用和我们下面要讲的Proxy是差不多的。

## 代理 Proxy

![image-20200608104342882](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/bMGhsKQUiCLl5Hf.png)

### 语法

```javascript
    let p = new Proxy(target, handler);
```

target：一个目标对象(可以是任何类型的对象，包括本机数组，函数，甚至另一个代理)用Proxy来包装。
handler：一个对象，其属性是当执行一个操作时定义代理的行为的函数。

### 代理的使用

**基础demo：**Proxy的demo有很多，我们只分析基础demo，主要看new Proxy({}, handler)的操作，指定目标obj对象，然后handler对象执行get()操作，get()返回值的判断是，如果name是target目标对象的属性，则返回target[name]的值，否则返回37，最后测试的时候，p.a是对象p的key，所以返回a的value，而p.b不存在，返回37。

```javascript
    const obj = {
      a: 10
    }
    let handler = {
        get: function(target, name){
            console.log('test: ', target, name)
            // test:  {"a":10} a
            // test:  {"a":10} b
            return name in target ? target[name] : 37
        }
    }
    let p = new Proxy(obj, handler)
    console.log(p.a, p.b) // 10 37
```

这个例子的作用是拦截目标对象obj，当执行obj的读写操作时，进入handler函数进行判断，如果读取的key不存在，则返回默认值。

我们使用一些http-proxy插件或者webpack的时候，有时候需要访问某个api时，跳转到指定的url，这种方式也能解决跨域访问。这种代理模式和Proxy的代理有异曲同工之妙。但是，别混为一体了。

```javascript
    module.exports = {
        devServer: {
           proxy: [
               {
                    context: "/api/*", //代理API
                    target: 'https://www.hyy.com', //目标URL
                    secure: false
              }
           ]
        }
    }
```

# 十三、用模块封装代码

## 模块的定义

模块是自动运行在严格模式下并且没有办法退出运行的JavaScript代码。

模块可以是函数、数据、类，需要指定导出的模块名，才能被其他模块访问。

```javascript
    //数据模块
    const obj = {a: 1}
    //函数模块
    const sum = (a, b) => {
      return a + b
    }
    //类模块
    class My extends React.Components {
    
    }
```

## 模块的导出

给数据、函数、类添加一个export，就能导出模块。一个配置型的JavaScript文件中，你可能会封装多种函数，然后给每个函数加上一个export关键字，就能在其他文件访问到。

```javascript
    //数据模块
    export const obj = {a: 1}
    //函数模块
    export const sum = (a, b) => {
      return a + b
    }
    //类模块
    export class My extends React.Components {
    
    }
```

## 模块的引用

在另外的js文件中，我们可以引用上面定义的模块。使用import关键字，导入分2种情况，一种是导入指定的模块，另外一种是导入全部模块。

1、导入指定的模块。

```javascript
    //导入obj数据，My类
    import {obj, My} from './xx.js'
    
    //使用
    console.log(obj, My)
```

2、导入全部模块

```javascript
    //导入全部模块
    import * as all from './xx.js'
    
    //使用
    console.log(all.obj, all.sun(1, 2), all.My)
```

## 默认模块的使用

如果给我们的模块加上default关键字，那么该js文件默认只导出该模块，你还需要把大括号去掉。

```javascript
    //默认模块的定义
    function sum(a, b) {
        return a + b
    }
    export default sum
    
    //导入默认模块
    import sum from './xx.js'
```

## 模块的使用限制

不能在语句和函数之内使用export关键字，只能在模块顶部使用，作为react和vue开发者的你，这个限制你应该很熟悉了。

**在react中，模块顶部导入其他模块。**

```javascript
    import react from 'react'
```

**在vue中，模块顶部导入其他模块。**

```javascript
    <script>
        import sum from './xx.js'
    </script>
```

## 修改模块导入和导出名

有2种修改方式，一种是模块导出时修改，一种是导入模块时修改。

1、导出时修改：

```javascript
    function sum(a, b) {
        return a + b
    }
    export {sum as add}

    import { add } from './xx.js'
    add(1, 2)
```

2、导入时修改：

```javascript
    function sum(a, b) {
        return a + b
    }
    export sum

    import { sum as add } from './xx.js'
    add(1, 2)
```

## 无绑定导入

当你的模块没有可导出模块，全都是定义的全局变量的时候，你可以使用无绑定导入。

模块：

```javascript
    let a = 1
    const PI = 3.1314
```

无绑定导入：

```javascript
    import './xx.js'
    console.log(a, PI)
```

## 浏览器加载模块

有用过webpack打包js模块的同学可能有经验，使用webpack打包了多个js文件，然后放到HTML使用script加载时，如果加载顺序不对，就会出现找不到模块的错误。

这是因为模块之间是有依赖关系的，就像你使用jQuery的时候，必须先加载jQuery的代码，才能使用jQuery提供的方法。

**加载模块的方法，总是先加载模块1，再加载模块2，因为module类型默认使用defer属性。**

```javascript
    <script type="module" src="module1.js"></script>
    <script type="module" src="module2.js"></script>eee
```