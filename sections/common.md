# 基础问题
* [`[Basic]` 1.变量类型](/sections/common.md#变量类型)
* [`[Basic]` 2.引用传递与值传递与指针](/sections/common.md#引用传递与值传递与指针)
* [`[Basic]` 3.作用域](/sections/common.md#作用域)
* [`[Basic]` 4.类型判断](/sections/common.md#类型判断)
* [`[Basic]` 5.内存释放](/sections/common.md#内存释放)
* [`[Basic]` 6.ES6新特性](/sections/common.md#ES6新特性)

## 变量类型
JavaScript中的变量有5个基本数据类型（Undefined, Null, Boolean, Number, String）和引用数据类型（Object，Function，Array等）。
```
类型        可能的值
Undefined   只有undefined，比如：var message; console.log(message == undefined);//true
Null        只有null
Boolean     true 和 false
Number      整数和浮点数，例如：1和3.14e10
String      任何字符串，例如： var a = 'string';
```
### 1.1基本类型和引用类型的区别
a.基本类型和引用类型的声明方式是一样的：
```
// 声明一个String类型的变量
var str = "string";

// 声明一个引用类型的变量，并添加属性
var person = new Object();
person.name = "Jeremy";
```
b.二者的主要区别在于对变量内容保存的方式，基本类型的变量中存储的就是简单的数据段，而引用类型变量存储的是指向对象的引用，比如：
```
// 基本类型的变量复制，可以看出基本类型变量存储的就是变量的值
var num1 = 1;
var num2 = num1;
console.log(num2); //1

// 引用类型的变量复制，可以看出引用变量中存储的是指向对象的引用
// Object对象存储于堆中，而obj1和obj2分别存储了指向此Object对象的引用
var obj1 = new Object();
obj1.name = "Jeremy";
var obj2 = obj1;
obj2.name = "James";
console.log(obj1.name); //James
```
## 引用传递与值传递与指针
“简单点说, 对象是引用传递, 基础类型是值传递, 通过将基础类型包装 (boxing) 可以以引用的方式传递”这是大部分人这样认为的，但我更赞同下面这种解释
### 2.1传值还是传引用
```
function changeStuff(num, obj1, obj2)
{
    num = num * 10;
    obj1.item = "changed";
    obj2 = {item: "changed"};
}

var num = 10;
var obj1 = {item: "unchanged"};
var obj2 = {item: "unchanged"};
changeStuff(num, obj1, obj2);
console.log(num);   // 10
console.log(obj1.item);    // changed
console.log(obj2.item);    // unchanged
```
以上面的`changeStuff(num, obj1, obj2)`为例，解释传值还是传引用
### 2.2传值
函数内的num, obj1, obj2都将是一份新的内存，与调用函数之前定义的三个变量毫无关系。函数内无论怎么修改这三个参数，外部定义的三个变量的值始终不变
传值的意思就是：传内存拷贝。

### 2.3传引用
函数内的num, obj1, obj2都分别指向一块内存，该内存就是调用函数之前定义的三个变量时创建的内存。函数内对这三个参数所做的任何改动，都将反映到外部定义的三个变量上。传引用的意思就是：传对地址的引用，传地址指向。

从上面的代码可以看出，JavaScript中函数参数的传递方式既不是传值，也不是传引用，而是叫[call-by-sharing](http://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_sharing)。


### 2.4Call-by-sharing
它的意思是：传引用的拷贝。
* `changeStuff()`中的num, obj1, obj2都是一个引用
* 他们的内容是某块内存的地址
* 这个地址的值来自于外部定义的三块内存，是那三块内存地址的一份拷贝
* 同时，在函数内部这三个参数的值是可以直接被修改的，可以指向其他对象（由于JavaScript中没有指针或引用运算符，只能直接修改）

因此，我们从内存和引用的角度再来看看`changeStuff()`的定义：

```
function changeStuff(num, obj1, obj2)
{
    num = num * 10; // 对num赋值，修改num的指向，新内存的内容为old_num * 10
    obj1.item = "changed";  // 修改原始obj1内存中的内容
    obj2 = {item: "changed"};   // 对obj2赋值，修改obj2指向，新内存的内容为{item: "changed"}
}
```

这里我们忽略`num = num * 10`具体是如何完成的。这依赖于具体语言的解释器，实际上包含了数个对指针的操作。

由于call-by-sharing本质上也是**传值**，因此，也可以说JavaScript的传参方式都是传值。总的来说，基本类型的参数传递复制的是具体的值，而引用类型的参数传递复制的是这个引用变量存储的对对象的引用。

为了进一步证明引用类型的参数传递是按值传递而不是按引用传递的，请看：
```
function setName(obj) {
	obj.name = "James";
	obj = new Object();
	obj.name = "Leon"
}

var person = new Object();
person.name = "Jeremy";
setName(person);
console.log(person.name);//James
```
以上代码输出的是James，如果是按引用传递，那么以上代码输出的是Leon。实际上，当执行obj.name = "James"的时候，引用所指向的对象的值已经发生了改变，当在对obj进行覆盖的时候，obj的值是一个指向局部对象的引用，而这个引用无法对外部的对象产生影响，并且此对象会在函数执行结束之后销毁。

除了JavaScript之外，Python, Java, Ruby, Scheme等语言也是采用call-by-sharing的求值策略。

### 2.5 module.exports和exports
明白了值传递和引用传递，这里插个小曲说明一下模块加载的问题。也是常问的问题，答案随便也能搜到，这里只是引用一段我认为解释得很到位的一段解释：
```
require 用来加载代码，而 exports 和 module.exports 则用来导出代码。但很多新手可能会迷惑于 exports 和 module.exports 的区别，为了更好的理解 exports 和 module.exports 的关系，我们先来巩固下 js 的基础。示例：

test.js

var a = {name: 1};
var b = a;

console.log(a);
console.log(b);

b.name = 2;
console.log(a);
console.log(b);

var b = {name: 3};
console.log(a);
console.log(b);

运行 test.js 结果为：

{ name: 1 }
{ name: 1 }
{ name: 2 }
{ name: 2 }
{ name: 2 }
{ name: 3 }

解释：a 是一个对象，b 是对 a 的引用，即 a 和 b 指向同一块内存，所以前两个输出一样。当对 b 作修改时，即 a 和 b 指向同一块内存地址的内容发生了改变，所以 a 也会体现出来，所以第三四个输出一样。当 b 被覆盖时，b 指向了一块新的内存，a 还是指向原来的内存，所以最后两个输出不一样。

明白了上述例子后，我们只需知道三点就知道 exports 和 module.exports 的区别了：
1.module.exports 初始值为一个空对象 {}
2.exports 是指向的 module.exports 的引用
3.require() 返回的是 module.exports 而不是 exports

我们经常看到这样的写法：

exports = module.exports = somethings
上面的代码等价于:

module.exports = somethings
exports = module.exports
原理很简单，即 module.exports 指向新的对象时，exports 断开了与 module.exports 的引用，那么通过 exports = module.exports 让 exports 重新指向 module.exports 即可。
```
官方的中文文档也解释得很明确：
```
function require(/* ... */) {
  const module = { exports: {} };
  ((module, exports) => {
    // 模块代码在这。在这个例子中，定义了一个函数。
    function someFunc() {}
    exports = someFunc;
    // 此时，exports 不再是一个 module.exports 的快捷方式，
    // 且这个模块依然导出一个空的默认对象。
    module.exports = someFunc;
    // 此时，该模块导出 someFunc，而不是默认对象。
  })(module, module.exports);
  return module.exports;
}
```

### 2.6参考链接
* [JavaScript中的参数传递](http://weizhifeng.net/arguments-of-function-in-JavaScript.html)
* [JS 中没有按地址（引用）传递，只有按值传递](http://www.cnblogs.com/youxin/p/3354903.html)
* [Call_by_sharing](https://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_sharing)
* [js-stackoverflow-highest-votes](https://github.com/simongong/js-stackoverflow-highest-votes/edit/master/questions21-30/parameter-passed-by-value-or-reference.md)
* [exports 和 module.exports 的区别](https://cnodejs.org/topic/5231a630101e574521e45ef8)


## 作用域

### 3.1函数作用域
1.在es5中声明变量的方式只有var或不使用关键字var。其中使用var关键字声明的变量都是函数作用域。如下：
```
    function rainman(){
        // rainman函数体内存在三个局部变量 i j k
        var i = 0;
        if ( 1 ) {
            var j = 0;
            for(var k = 0; k < 3; k++) {
                console.log( k );  //分别输出 0 1 2
            }
            console.log( k );        //输出3
        }
        console.log( j );            //输出0
    }
```
上面的例子中，说明i、j、k的作用域都是在函数rainman。var定义的不存在块级作用域，但却存在变量提升。如下：
```
     function test() {
       console.log(a); // 输出undefined
       var a = 1;
     }
```
尽管a的声明语句 ``` var a = 1```在a被使用之后才声明，但对于js来说却不会报错，就是存在变量提升的缘故。上面的代码等价于：
```
     function test() {
       var a;
       console.log(a); // 输出undefined
       a = 1;
     }
```
这即js所谓的变量提升。
2.变量查找规则。js在使用变量时，会从最里层的函数作用域开始往外找。找到同名的变量则会立马返回，而不会继续往外层找。如果没找到同名变量，则会继续往外层函数继续查找，如果都找不到，再到全局变量里面找，直到找到。找不到则报错“ReferenceError： XXX is not defined”。
```
     function outTest() {
       console.log(a); // 输出undefined
       var a = 1;
       function innerTest() {
          console.log(a);//同样输出undefined，下面那行代码会出现变量提升，导致查找a变量时，在innerTest就找到了，所以直接返回，而不会打印1
	  var a = "a is not been defined here but been assigned here"
       }
     }
```
  上面的例子也说明内嵌函数可以使用外层函数的变量，但外层函数却不能直接使用内嵌函数的变量。
  
### 3.2块级作用域
从es6开始，js就已经开始支持块级作用域。阮一峰大神的[ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/let)已经说得很清楚了,这里就不赘述了。
## 类型判断

## 内存释放

## ES6新特性

### ES6声明变量的六种方法
* var (es5也有)
* function (es5也有)
* let (es6新特性)
* const (es6新特性)
* import (es6新特性)
* class (es6新特性)
