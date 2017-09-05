# 基础问题
* [`[Basic]` 变量类型](/sections/common.md#变量类型)
* [`[Basic]` 类型判断](/sections/common.md#类型判断)
* [`[Basic]` 作用域](/sections/common.md#作用域)
* [`[Basic]` 引用传递与值传递与指针](/sections/common.md#引用传递与值传递与指针)
* [`[Basic]` 内存释放](/sections/common.md#内存释放)
* [`[Basic]` ES6新特性](/sections/common.md#ES6新特性)

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
### 基本类型和引用类型的区别
1.基本类型和引用类型的声明方式是一样的：
```
// 声明一个String类型的变量
var str = "string";

// 声明一个引用类型的变量，并添加属性
var person = new Object();
person.name = "Jeremy";
```
2.二者的主要区别在于对变量内容保存的方式，基本类型的变量中存储的就是简单的数据段，而引用类型变量存储的是指向对象的引用，比如：
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
## 类型判断
“简单点说, 对象是引用传递, 基础类型是值传递, 通过将基础类型包装 (boxing) 可以以引用的方式传递”这是大部分人这样认为的，但我更赞同下面这种解释
### 传值还是传引用
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
#### 传值
函数内的num, obj1, obj2都将是一份新的内存，与调用函数之前定义的三个变量毫无关系。函数内无论怎么修改这三个参数，外部定义的三个变量的值始终不变
传值的意思就是：传内存拷贝。

### 传引用
函数内的num, obj1, obj2都分别指向一块内存，该内存就是调用函数之前定义的三个变量时创建的内存。函数内对这三个参数所做的任何改动，都将反映到外部定义的三个变量上。传引用的意思就是：传对地址的引用，传地址指向。

从上面的代码可以看出，JavaScript中函数参数的传递方式既不是传值，也不是传引用，而是叫[call-by-sharing](http://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_sharing)。


### Call-by-sharing
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

### 参考链接
* [JavaScript中的参数传递](http://weizhifeng.net/arguments-of-function-in-JavaScript.html)
* [JS 中没有按地址（引用）传递，只有按值传递](http://www.cnblogs.com/youxin/p/3354903.html)
* [Call_by_sharing](https://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_sharing)
* [js-stackoverflow-highest-votes](https://github.com/simongong/js-stackoverflow-highest-votes/edit/master/questions21-30/parameter-passed-by-value-or-reference.md)


## 作用域

## 引用传递与值传递与指针

## 内存释放

## ES6新特性
