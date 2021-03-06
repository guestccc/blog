# 面试准备
<!-- bind 函数，写个原型继承什么的
全是基础，跨域，闭包之类的。
跨域，继承，多态，面向对象，原型链，设计模式，http 请求。
面相对象，前端跨域，闭包 -->

## http状态码

```
200 
204 (无内容)

301 永久重定向

304 未修改

400 服务器不理解请求的语法

401 请求登录认证

404 (未找到) 服务器找不到请求的网页。

412 未满足请求者设置的条件

500 服务器错误
```

## 跨域

浏览器同源策略

> 同源政策的目的，是为了保证用户信息的安全，防止恶意的网站窃取数据。

协议://域名:端口号

### 限制

（1） Cookie、LocalStorage 和 IndexDB 无法读取。

（2） DOM 无法获得。

（3） AJAX 请求不能发送。

### 一、cookie、localStorage

#### 1. 一级域名相同

* 设置document.domain

    cookie取值

    iframe拿dom

* Set-Cookie: key=value; domain=.example.com; path=/

  服务器设置cookie


#### 2. 完全跨域

* 片段识别符

#

* window.name

窗口从非同源跳同源

* window.postMessage

a
```js
var popup = window.open('http://bbb.com', 'title');
popup.postMessage('Hello World!', 'http://bbb.com');
```
b
```js
window.addEventListener('message', function(e) {
  console.log(e.data);
},false);

event.source：发送消息的窗口
event.origin: 消息发向的网址
event.data: 消息内容
```

### 二 ajax

 - jsonp

 只有get请求，设置回调函数，携带函数名

 - WebSocket

 - CORS

    设置origin *

    简单


    非简单

    默认不到cookie
    除非开启Credentials



所有引用类型的值都是 Object 的实例

### 执行环境（执行上下文）

在 web 中，全局执行环境被认为是 window

每个函数都有自己的执行环境

每个环境都会有一个变量对象 每个变量对象都有一个作用域链

如果这个环境是函数，则将其活动对象(activation object)作为变量对象
每个环境都 

可以向上搜索作用域链，以查询变量和函数名

加长作用域，try-catch 中的 catch、 with 语句
这两个语句都会在作用域链的前端添加一个变量对象

#### 变量对象 活动对象

[红宝书（p73）]()：每个执行环境都有一个 与之关联的变量对象(variable object)，环境中定义的所有变量和函数都保存在这个对象中。

> 也就是说当前执行上下文中所有的变量和函数声明，都在这个变量对象中

```js
<execution context stock>:[
  <execution context>:{
    VO:{
      agruments:{
        0:1,
        length
      },
      a:uninit,//let const
      b:undefined,//var
      c:reference to function c(){},,//function
    }
  }
]
```

|变量|声明方式|vo 状态|解释|
|--|--|--|--|
|agruments|--| 已经初始化 有值/undefined |有传就有，没有就是undefined|
|a|let、const|uninit |未初始化|
|b|var|undefined |已经初始化，是个 undefined|
|c|function|reference to function c(){} |已经拿到这个函数的引用|

## 作用域


```js
function foo(val){
  let a = 1
  var b = 1
  function c () {

  }
}s
foo(1)
```

```js
1. foo定义，保存父级`作用域链`给 函数 的[[scope]]属性
foo.[[scope]] = [
  globalContext.VO
]

2. 执行上下文 入 栈
<execution context stock>.push(fooContext)

<execution context stock>:[
  <global execution context>,
  <foo execution context>:{
  }
]

3. 复制作用域链

<execution context stock>:[
  <global execution context>,
  <foo execution context>:{
    <scope chain>:foo.[[scope]] //[globalContext.VO]
  }
]

4. 创建活动对象
<execution context stock>:[
  <global execution context>,
  <foo execution context>:{
    AO:{
      agruments:{
        0:1,
        length
      },
      a:uninit,//let const
      b:undefined,//var
      c:reference to function c(){},,//function
    },
    <scope chain>:foo.[[scope]] //[globalContext.VO]
  },
]

5. 给当前执行上下文的作用域链，压入当前活动对象
<execution context stock>:[
  <global execution context>,
  <foo execution context>:{
    AO:{
      agruments:{
        0:1,
        length
      },
      a:uninit,//let const
      b:undefined,//var
      c:reference to function c(){},,//function
    },
    <scope chain>:[
      fooContext.AO,
      globalContext.VO
    ]
  },
]

6. 开始执行，修改活动对象🎡属性
<execution context stock>:[
  <global execution context>,
  <foo execution context>:{
    AO:{
      agruments:{
        0:1,
        length
      },
      a:1,//let const
      b:1,//var
      c:reference to function c(){},,//function
    },
    <scope chain>:[
      fooContext.AO,
      globalContext.VO
    ]
  },
]

7. c定义，保存父级作用域链给 c函数 的[[scope]]属性
c.[[scope]] = [
  fooContext.AO,
  globalContext.VO
]

8. foo执行上下文出栈
<execution context stock>:[
  <global execution context>
]
```

## 作用域链

作用域链本质上是一个指向变量对象的指针列表

## 执行栈和作用域链区分

> 作用域链，根据书写顺序(函数定义顺序)

> 执行上下文，根据执行顺序

1. 🌰

```js
function a (){
  b()
}
function b (){
  c()
}
function c (){
}
a()

作用域：
a['global scope','a scope']
b['global scope','b scope']
b['global scope','c scope']
执行栈：
ECS.push('ec global')
ECS.push('ec a')
ECS.push('ec b')
ECS.push('ec c')
ECS.pop()
ECS.pop()
ECS.pop()
ECS.pop()
```

2. 🌰🌰

```js
var aa = 'global'
function a (){
  var aa = 'aa'
  function b (){
    console.log(aa) //aa
    function c (){
    }
    return c()
  }
  return b()
}
a()

作用域：
global['global scope']
a['global scope','a scope',]
b['global scope','a scope','b scope']
c['global scope','a scope','b scope','c scope']

执行栈：
ECS.push('ec global')
ECS.push('ec a')
ECS.push('ec b')
ECS.push('ec c')
ECS.pop()
ECS.pop()
ECS.pop()
ECS.pop()
```

> return 的是执行后的结果，也就是说先执行，再 return，先入栈，入完再出

3. 🌰🌰🌰

```js
var aa = 'global'
function a (){
  var aa = 'aa'
  function b (){
    console.log(aa) //aa
    function c (){
    }
    return c
  }
  return b
}
a()()()

作用域：
global['global scope']
a['global scope','a scope',]
b['global scope','a scope','b scope']
c['global scope','a scope','b scope','c scope']

执行栈：
ECS.push('ec global')
ECS.pop()
ECS.push('ec a')
ECS.pop()
ECS.push('ec b')
ECS.pop()
ECS.push('ec c')
ECS.pop()

```

## 闭包

闭包是指有权访问另一个 函数作用域中的变量的函数

mdn： 闭包是指那些能够访问自由变量的函数。

### 理论上

《JavaScript权威指南》中就讲到：从技术的角度讲，所有的JavaScript函数都是闭包。

闭包 = 函数 + 函数能够访问的自由变量
> 自由变量是指在函数中使用的，但既不是函数参数也不是函数的局部变量的变量。

从理论角度：所有的函数。因为它们都在创建的时候就将上层上下文的数据保存起来了。哪怕是简单的全局变量也是如此，因为函数中访问全局变量就相当于是在访问自由变量，这个时候使用最外层的作用域。

### 实践上

ECMAScript中，闭包指的是：


从实践角度：以下函数才算是闭包：

即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）
在代码中引用了自由变量

## 原型链

## new 实质

四部曲

1. 创建一个对象 x
2. x._proto_ 指向 class.prototype
3. 指定 this
4. 如果函数有基本类型的返回值，则返回对象 x，如果函数有引用类型的返回值，则返回该引用类型

模拟 new

```js
function class (fn,...val){
  var o = {

  }
  o.__proto__ = fn.prototype
  let a = fn.apply(o,val)
  return typeof a === 'object'|| typeof a === 'function'?a:o
}
```

## this


> 粗浅层
默认绑定

隐式绑定

显示绑定

new

回调函数

> 深入

1. expression 表达式 《javaScript权威指南》

调用表达式

如果调用表达式之前的表达式，是一个属性访问表达式，那么作为属性访问主题的对象和数组便是其调用方法内this的指向

2. 引用法 阮一峰老师

[运算符顺序](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)


## apply call bind

```js
var obj = {
  value:1,
}

function foo(who1,who2){
  console.log(this.value)
  console.log(who1+'fuck'+who2)
}


Function.prototype.call2 = function(context,...val){
  context = context || window
  context.foo2 = this
  var arr = []
  for(let i = 0;i<val.length;i++){
    arr.push('val['+i+']')
  }
  return eval('context.foo2('+arr+')')
}

foo.call2(obj,'ccc','ccc2')

// 加闭包，模拟用apply
Function.prototype.bind2 = function(context,...val1){
  self = this
  return function (...val2){
    return self.apply(context,[...val1,...val2])
  }
}

b = foo.bind2(obj)
b('ccc','ccc2')

// 适配 new
Function.prototype.bind2 = function (context) {

    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```

### call

```js
Function.prototype.call2 = function (context){
  context = context || window
  context._fn = this
  var arr = []
  Object.keys(arguments).forEach((item) => {
    if(!index) return
    arr.push(arguments[item])
  })
  var result = eval('context._fn('+arr+')')
  delete context._fn
  return result
}
```


### apply

```js
Function.prototype.apply = function (context, arr) {
    var context = Object(context) || window;
    context.fn = this;

    var result;
    if (!arr) {
        result = context.fn();
    }
    else {
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        result = eval('context.fn(' + args + ')')
    }

    delete context.fn
    return result;
}
```

### bind

```js
Function.prototype.bind2 = function (context){
  var fn = this
  var arr = Array.prototype.slice.call(arguments,1)
  return function (){
    return fn.apply(context,arr.concat(Array.prototype.slice.call(arguments)))
  }
}

var  obj ={
  name:'ccc'
}
function test(age,tell){
  return this.name + age + tell
}
```

## 面对对象

封装
继承
多态
设计模式
http
跨域

## 封装、继承、多态

> 造车：写构造函数啊、写函数啊、定义属性啊

> 开车：调用函数、new Class()

### 一、封装

> 封装：`1`.把客观事物封装成抽象的类 `2`.隐藏属性和方法的实现细节 `3`.仅对外公开接口。

#### 1. js 中的 private

因为 javascript 函数级作用域的特性（在函数中定义的属性和方法外界访问不到），所以我们在函数内部直接定义的属性和方法都是私有的。

> 由于函数作用域的特性，函数内部定义的变量外部访问不到，是私有的

#### 2. js 中的 public

通过 new 关键词实例化时，this 定义的属性和变量都会被复制一遍，所以通过 this 定义的属性和方法就是公有的。通过 prototype 创建的属性在类的实例化之后类的实例化对象也是可以访问到的，所以也是公有的。

> 构造函数通过`new`实例化的时候，函数内部的 this 指向了实例对象，通过这种方式定义的属性和函数就是共有的，外部可以访问的

> 通过 prototype 创建的属性和函数，构造函数通过`new`实例化后，把 prototype 对象的地址赋给了实例化对象的`__proto__`对象，同样，外部也可以访问到

#### 3. js 中的 protected

在函数的内部，我们可以通过 this 定义的方法访问到一些类的私有属性和方法，在实例化的时候就可以初始化对象的一些属性了。

> 通过构造函数内部 this 创建一个函数，返回私有变量，外部只有通过调用这个函数才能拿到这个变量，受保护的

#### 4. new 的实质

**4 步**

1. `new xxClass()`新建了一个对象

2. `obj.__proto__` 地址指向了 `xxClass.prototype`

3. `xxClass`构造函数的 this 绑定到新建的对象

4. 默认返回这个对象，如果函数又返回值;如果构造函数的返回值为基本数据类型`string`,`boolean`,`number`,`null`,`undefined`,那么就返回新对象;如果构造函数的返回值为对象类型，那么就返回这个对象类型

#### 5. 三个特性

1. 构造函数`this`

2. 构造函数`prototype`

3. 构造函数`.`

### 二、继承

首先，继承有几种方式，例如，1. 原型继承 2.借用构造函数继承 3. 组合继承 4.寄生继承 。。。

原型继承涉及函数 prototype 原型链的这个概念，和 new 的实质

借用构造函数继承 通过函数内部直接调用构造函数

继承其实就是通过 构造函数`this`，构造函数`prototype`，构造函数`.` ,这三种特性来实现的

> 继承：从一般到特殊的过程，子类继承父类的所有功能，并进行拓展

几种方式

1. [类式继承](<####类式(原型)继承>)
2. [构造函数继承](####构造函数继承)
3. [构造函数继承](####构造函数继承)
4. [构造函数继承](####构造函数继承)

#### 1. 类式(原型)继承

```js
// 造车
function Baba(name){
  // 属性
  this.babaName = name
  // 小动作
  this.fangpi = function (){
    return '我爱放屁'
  }
}
// 原型小动作   通过new才能继承
Baba.prototype.getName = function (){
  return this.babaName
}


// 造车
function Erzi (name){
  this.erziName = name
}
// 原型小动作   通过new才能继承
Erzi.prototype = new Baba('ccc')
// Erzi.prototype = new Baba()


// 开车
let ni = new Erzi('ccc的儿子')
console.log(ni.babaName) // ccc

炸了，老爸的名字怎么总是ccc
```

特点：

- 通过把父类实例子类原型 `prototype` 对象

缺陷：

- 如果父类通过 this 定义了一个引用类型的变量，那么这个引用类型的变量就会被所有子类的实例所共有，如果在一个实例中更改改引用类型变量地址指向的内容，所有实例都会被影响
- 父类的实例化操作不够灵活

#### 2. 构造函数继承

```js
// 造车
function Baba(name){
  // 属性
  this.babaName = name
  // 小动作
  this.fangpi = function (){
    return '我爱放屁'
  }
}
// 原型小动作   通过new才能继承
Baba.prototype.getName = function (){
  return this.babaName
}


// 造车
function Erzi (name,babaName){
  this.erziName = name
  Baba.call(this,babaName) // 这里的bind和apply和call也是一个知识点
}


// 开车
let ni = new Erzi('ccc的儿子','ccc')
console.log(ni.babaName) // ccc

炸了，老爸的原型去哪里

```

特点：

- 通过 call，apply 和 bind，来替代 new 过程中绑定 this 的操作

缺点：

- 父类不是通过`new`去调用，所以没继承到父类的原型`prototype`
- 每次生产一个子类实例，都会执行一次父类函数，消耗内存

|              | 特点                                                   | 优点                                            | 缺点                                                                                        |
| ------------ | ------------------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------------------------------------------- |
| 类式继承     | 父类实例化子类的原型 prototype 对象                    | 子类的实例化对象可以拿到父类的实例对象和原型    | 子类相互影响(因为父类只实例化了一次 )                                                       |
| 构造函数继承 | 通过 call、apply、bind 替代 new 过程中绑定 this 的操作 | 子类不相互影响(子类实例化的时候，父类也调用了 ) | 1. 每次生产一个子类实例，都会执行一次父类函数，消耗内存<br>2.没有继承到父类的原型 prototype |

### 组合式继承

```js
// 造车
function Baba(name) {
  // 属性
  this.babaName = name
  // 小动作
  this.fangpi = function() {
    return '我爱放屁'
  }
}
// 原型小动作   通过new才能继承
Baba.prototype.getName = function() {
  return this.babaName
}

// 造车
function Erzi(name, babaName) {
  this.erziName = name
  Baba.call(this, babaName) // 拿到this赋值的属性
}

Erzi.prototype = new Baba() // 拿到父类的原型prototype
// 开车
let ni = new Erzi('ccc的儿子', 'ccc')
```

> 重点，对象属性的访问操作，也就是左赋值

特点：

- 通过 call，apply 和 bind，来替代 new 过程中绑定 this 的操作


## 浏览器缓存策略；

## HTTP协议 前端安全机制