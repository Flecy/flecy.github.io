---
title: JavaScript中的this
toc: true
date: 2020-05-05 00:53:38
tags: js
categories:
- 前端
- 基础
---

## this是什么
this是一个很特别的关键字，它是在函数被调用时发生的绑定，并不是在编写时进行绑定，它的上下文取决于函数调用时的各种条件。this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。

<!--more-->

当一个函数被调用时，会创建一个活动记录(有时也称为执行上下文)。这个记录会包含函数在哪里被调用(调用栈)、函数的调用方式、传入的参数等信息。this就是这个记录的一个属性，会在函数执行的过程中用到。


## this绑定规则
在理解this的绑定过程之前，首先要理解调用位置：调用位置就是函数在代码中被调用的位置。最重要的就是分析调用栈(为了到达执行位置所调用的所有函数)。调用位置就在当前正在执行的函数的前一个调用中。

```javascript
function baz() {
	console.log("baz");
	bar();
}

function bar() {
	console.log("bar");
	foo();
}

function foo() {
	debugger;
	console.log("foo");
}

baz();
```
![foo函数的调用栈](http://image.manylu.com/js-this-1.png)
如果想要分析this的绑定，使用开发工具得到调用栈，栈中的第二个元素即为真正的调用位置。上图foo函数的调用位置为bar函数中。

### 默认绑定
最常用的函数调用类型：独立函数调用。可以把这条规则看作是无法应用其他规则时的默认规则。

非严格模式下，默认绑定规则将this绑定到全局对象上。
严格模式下，不能将全局对象用于默认绑定，this会绑定到undefined。
```javascript
//非严格模式下
function foo() {
	var a = 1;
	console.log(this.a);
}

var a = 2;

foo(); //2  全局对象上的变量a

```

```javascript
//严格模式下
"use strict";

function foo() {
	console.log("this: ", this);
}

foo(); //"this: undefined"
```

### 隐式绑定
当函数引用有上下文对象时，隐式绑定规则会把函数调用中的this绑定到这个上下文对象。
```javascript
function foo() {
	var a = 0;
	console.log(this.a);
}

var obj = {
	a: 1,
	foo: foo
};

obj.foo(); //1  调用foo()时this被绑定到obj上
```

被隐式绑定的函数有时会丢失绑定对象，应用默认绑定，从而将this绑定到全局对象或者undefined上(取决于是否是严格模式)
```javascript
function foo() {
	var a = 0;
	console.log(this.a);
}

var obj = {
	a: 1,
	foo: foo
};

var bar = obj.foo; //函数别名
var a = 2;

bar(); //2
//虽然bar是obj.foo的一个引用，但是实际上它引用的是foo函数本身
//此时的bar()其实是一个不带任何修饰符的函数调用，因此应用默认绑定
```
回调函数丢失this绑定也是很常见的，无论哪种情况，this的改变都是意想不到的。
```javascript
function foo() {
	var a = 0;
	console.log(this.a);
}

var obj = {
	a: 1,
	foo: foo
};

var a = "global a";

setTimeout(obj.foo, 1000);//1秒后执行函数obj.foo
// "global a"

/*
JavaScript环境中内置的setTimeout()函数实现和下面伪代码类似：
function setTimeout(fn, delay) {
	//等待delay秒
	fn();
}
*/

//参数传递其实是一种隐式赋值，这里相当于fn = obj.foo,所以结果同上例
```

隐式绑定可理解为我们必须在一个对象内部包含一个指向函数的属性，并通过这个属性间接引用函数，从而将this间接(隐式)绑定到这个对象上。

### 显式绑定
如果我们不想在对象内部包含函数引用，而想在某个对象上强制调用函数，则可以使用显示绑定。具体点就是使用函数的call(...)和apply(...)方法。

Function.prototype.call([thisArg[, arg1, arg2,...argN]]);
Function.prototype.apply(thisArg, [ argsArray]);
这两个方法的thisArg参数就是给this准备的，在调用函数时将this直接绑定到thisArg上。因为我们可以直接指定this的绑定对象，因此我们称之为显式绑定。
```javascript
var person1 = {
	fullName: function() {
		return this.firstName + " " + this.lastName; 
	}
}

var person2 = {
	firstName: "Lucy",
	lastName: "Wang"
};

person1.fullName.call(person2); // "Lucy Wang"
```
但是显式绑定仍然无法解决我们之前提出的丢失绑定的问题。显示绑定的一个变种-硬绑定则可以解决
```javascript
function foo() {
	var a = 0;
	console.log(this.a);
}

var obj = {
	a: 1,
	foo: foo
};

var a = "global a";

var bar = function() {
	foo.call(obj);
};

bar(); //1
bar.call(window); //1
setTimeout(bar, 1000); //1

/*
创建了函数bar(),在它内部手动调用foo.call(obj),强制的把foo的this绑定到了obj
无论之后如何调用函数bar，它总会手动在obj上调用foo
这种绑定是一种显式的强制绑定，称之为硬绑定
*/
```
由于硬绑定是一种非常常用的模式，所以ES5提供了内置的方法Function.prototype.bind

bind()方法创建一个新的函数，在bind()被调用时，这个新函数的this被指定为bind()的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

### new绑定
在传统的面向类的语言中，"构造函数"是类中的一些特殊方法，使用new初始化类时会调用类中的构造函数。JS中也有一个new操作符，但是JS中new的机制实际上和面向类的语言完全不同。

在JavaScript中，构造函数只是一些使用new操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类。实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被new操作符调用的普通函数而已。

包括内置对象函数在内的所有函数都可以用new来调用，这种函数调用被称为构造函数调用。
实际上并不存在所谓的"构造函数"，只有对函数的"构造调用"

使用new来调用函数，或者说发生构造函数调用时，会自动执行下面的操作：
- 创建(或者说构造)一个全新的对象
- 这个新对象会被执行[[Prototype]]连接
- 这个新对象会被绑定到函数调用的this
- 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象

```javascript
function Person(name) {
	this.name = name;
}

var person = new Person("Lucy");

console.log(person.name); //"Lucy"
```


## this绑定优先级
我们已经了解了函数调用中this绑定的四条规则，只需要找到函数的调用位置并判断应有哪条规则就行。但是，如果某个调用位置可以应用多条规则的话，则必须给这些规则设定优先级。

默认绑定的优先级是四条规则中最低的。

### 隐式绑定vs显式绑定
```javascript
function foo() {
	console.log(this.a);
}

var obj1 = {
	a: 1,
	foo: foo
};

var obj2 = {
	a: 2,
	foo: foo
};

//隐式绑定
obj1.foo(); //1
obj2.foo(); //2

//运用了隐式绑定and显式绑定
obj1.foo.call(obj2); //2
obj2.foo.call(obj1); //1
```
显式绑定优先级高于隐式绑定

### new绑定vs隐式绑定
```javascript
function foo(something) {
	this.a = something;
}

var obj1 = {
	foo: foo
};

obj1.foo(1);//隐式绑定
console.log(obj1.a); //1

var bar = new obj1.foo(2); //运用了new绑定and隐式绑定
console.log(obj1.a); //1
console.log(bar.a); //2
```
new绑定比隐式绑定优先级高

### new绑定vs硬绑定
```javascript
function foo(something) {
	this.a = something;
}

var obj1 = {};

var bar = foo.bind(obj1); //硬绑定 bar被硬绑定到obj1上
bar(2);
console.log(obj1.a); //2

var baz = new bar(3); //应用了硬绑定的基础上再应用new绑定
console.log(obj1.a); //2
console.log(baz.a); //3 new绑定修改了硬绑定到obj1上的bar函数调用中的this
```

new绑定比硬绑定优先级高

### 优先级小结
可以根据优先级来判断函数在某个位置应用的是哪条规则。可以按照下面的顺序进行判断：
1. 函数是否在new中调用(new绑定)？如果是的话this绑定的就是新创建的对象。
2. 函数是否通过call、apply(显式绑定)或硬绑定调用？如果是的话，this绑定的是指定的对象。
3. 函数是否在某个上下文对象中调用(隐式绑定)？如果是的话，this绑定的是那个上下文对象。
4. 如果都不是的话，使用默认绑定。在严格模式下，绑定到undefined,否则绑定到全局对象。

## 参考内容
- 《你不知道的JavaScript上卷》第二部分
- [MDN Function.prototype.call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
- [MDN Function.prototype.apply()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)