---
title: JavaScript中的属性描述对象
toc: true
date: 2020-05-01 08:24:23
tags: js
categories:
- 前端
- 基础
---

## 简述
JavaScript提供了一个内部数据结构，用来描述对象的属性，控制它的行为，比如该属性是否可写、可遍历等等。这个内部数据结构称为"属性描述对象"。每个属性都有自己对应的属性描述对象，保存该属性的一些元信息。
<!--more-->

属性描述对象共提供6个元属性
1. value：属性的属性值，默认为undefined
2. writable：布尔值，表示属性值value是否可写，默认为true
3. enumerable：布尔值，表示该属性是否可遍历，默认为true
4. configurable：布尔值，表示可配置性，控制属性描述对象的可写性，默认为true
5. get：是一个函数，表示该属性的取值函数(getter)，默认为undefined
6. set：是一个函数，表示该属性的存值函数(setter)，默认为undefined

ECMA-262定义这些元属性是为了实现JavaScript引擎用的，因此在JavaScript中不能直接访问它们，为了表示它们是内部值，将它们放在两对方括号中。(e.g [[configurable]])

ECMAScript中有两种属性：
- 数据属性
- 访问器属性

## 数据属性
数据属性包含一个数据值的位置。在这个位置可以读取和写入值，数据属性有4个描述其行为的元属性
- [[configurable]]
- [[enumerable]]
- [[writable]]
- [[value]]

```javascript
let person = {
	age: 1
};

//获取person对象的age属性的属性描述对象
console.log(Object.getOwnPropertyDescriptor(person, "age")); 
/*
{
	configurable: true,
	enumerable: true,
	value: 1,
	writable: true
}
*/

Object.defineProperty(person, "age", {
	value: 2,
	writable: false, //不可写
	configurable: true,
	enumerable: true
});

person.age = 3; //非严格模式下忽略写入，严格模式下出现TypeError
console.log(person.age); //2  
```

只要属性是可配置的，即configurable为true的话，即可使用defineProperty()方法来修改属性描述符
```javascript
let obj = {
	a: 0
};

Object.defineProperty(obj, "a", {
	configurable: false //不可配置
});

obj.a = 1;
console.log(obj.a); //1  不可配置指的是不可配置属性描述符

Object.defineProperty(obj, "a", {
	configurable: true
}); // TypeError  
//不管是否处于严格模式，把configurable修改为false是一个单向操作，尝试修改一个不可配置的属性描述符都会出错

Object.defineProperty(obj, "a", {
	writable: false
}); //但是，即便configurable: false,还是可以将writable的状态由true改为false  但是无法由false改为true

Object.defineProperty(obj, "a", {
	writable: true //TypeError
}); 

delete obj.a; //false  除了无法修改，configurable: false还会禁止删除这个属性
```

enumerable这个描述符控制的是属性是否会出现在对象的属性枚举中
```javascript
let person = {
	age: 23,
	name: "Lucy",
	female: true
};

for (let attr in person) {
	console.log(attr);
};
//依次打印 age name female

Object.defineProperty(person, "name", {
	enumerable: false
});

for (let attr in person) {
	console.log(attr);
};
//依次打印 age female
```

## 访问器属性
访问器属性不包含数据值，它们包含一对getter和setter函数(非必需)

在读取访问器属性时，会调用getter函数；在写入访问器属性时，会调用setter函数并传入新值，这个函数决定如何处理数据
访问器属性有4个描述其行为的元属性
- [[configurable]]
- [[enumerable]]
- [[get]]
- [[set]]

访问器属性不能直接定义，必须使用Object.defineProperty()定义
```javascript
let person = {
	birth: 1996,
	_year: 2019, //下划线是一种常用的记号，用于表示只能通过对象方法访问的属性
	age: 23
};

Object.defineProperty(person, "year", {
	get: function() {
		return this._year;
	},
	set: function(newYear) {//访问器属性的常见方式即设置一个属性的值会导致其它属性发生变化
		this._year = newYear;
		this.age = newYear - this.birth;
	}
});

person.year = 2024;
console.log(person.age); //28
```
不一定非要同时指定getter和setter，只指定getter意味着属性是不能写，尝试写入属性会被忽略。在严格模式下，尝试写入只指定了getter函数的属性会抛出错误。类似的，只指定setter函数的属性不能读，否则在非严格模式下返回undefined，而在严格模式下会抛出错误。

```javascript
"use strict";
let person = {
	birth: 1996,
	_year: 2019, //下划线是一种常用的记号，用于表示只能通过对象方法访问的属性
	age: 23
};

Object.defineProperty(person, "year", {
	get: function() {
		return this._year;
	}
});

person.year = 2024;  //TypeError
```

## 参考内容
- [阮一峰JavaScript教程-属性描述对象](https://wangdoc.com/javascript/stdlib/attributes.html)
- 《你不知道的JavaScript上卷》第二部分 第三章
- 《JavaScript高级程序设计第三版》 第6章
