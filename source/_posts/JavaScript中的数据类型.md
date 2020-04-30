---
title: JavaScript中的数据类型
toc: true
date: 2020-04-28 18:12:54
tags:  js
categories: 
- 前端
- 基础
---

## 数据类型分类
JavaScript中共有8种数据类型
1. number
2. bigint
3. string
4. boolean
5. null
6. undefined
7. symbol
8. object
<!--more-->

## 数据类型简述
### 基本数据类型
#### number
数值类型，可以表示整数或者小数。除了常规的数字外，还包括"特殊数值"：Infinity、-Infinity、NaN
```javascript
var num1 = 1;
var num2 = 1.3;
alert(1/0); //Infinity
alert(Infinity); //Infinity
alert("hhh"/1); //NaN NaN表示一个计算错误。作用是为了使得数学运算是安全的，从而脚本不会因为错误的运算而停止
typeof num1; //"number"
```

#### string
字符串类型，必须放在引号中
1. 双引号："Flecy"
2. 单引号： 'Flecy'
3. 反引号： \`Flecy\`
单引号双引号无区别，反引号是功能扩展引号，允许将变量和表达式包装在${...}中，然后嵌入到字符串中。

```javascript
var name = "Flecy";
alert(`Hi, ${name}`); // "Hi, Flecy"

alert(`${1+2}`); //3
```

#### boolean
只包含两个值：true和false

```javascript
var result = true;
typeof result; //"boolean"
```

#### null
只含null一个值

```javascript
var result = null;
typeof result; //"object"  
```
null不是一个object,null有自己的类型，它就是一个特殊值，typeof null的结果为"object"是JavaScript语言的一个错误，为了兼容性保留下来了。

#### undefined
只含undefined一个值,undefined的含义是未被赋值。
如果一个变量已经被声明，但没有赋值，那么它的值就是undefined。

```javascript
var result;
typeof result; //"undefined"
```


### 复杂数据类型
#### object
各种值组成的集合，用于存储数据集合和更复杂的实体
```javascript
var blog = {
	name: "弗蕾西",
	domain: "manylu.com",
	owner: "Flecy",
	age: 1
};

typeof blog; // "object"
```



### 新增加的数据类型
不太想把这两个数据类型放在基本数据类型中，也算是为了更好的区分JS的变化吧，所以单独把它们放在第三类中。 


#### symbol
用于创建对象的唯一标识符
因为ES5的对象属性名都是字符串，为了保证每个属性的名字都是独一无二的，从根本上防止属性名的冲突，引入了symbol类型
symbol值通过Symbol函数生成
```javascript
var s = symbol();
typeof s; // "symbol"
```
Symbol函数前不能使用new命令，否则会报错，因为生成的symbol是一个原始类型的值，不是对象。

#### bigint
用于表示任意长度的整数，在JavaScript中，"number"类型无法代表大于2^53(或小于-2^53)的整数。
通过将n添加到整数字段末尾来创建bigint类型。
```javascript
var bigInt = 123456789n;
typeof bigInt; // "bigint"
```
**存在兼容性问题：Firefox和Chrome已经支持bigint了，但是Safari/IE/Edge还没有**

## typeof运算符
typeof运算符可以返回一个值的数据类型
除了上面代码中返回的类型外，函数也会特别返回
```javascript
typeof alert; // "function"
typeof []; // "object"
typeof {}; // "object"
```
JavaScript中没有一个特别的"function"类型，函数隶属于object类型，但是typeof会对函数区别对待
typeof区分不了对象的细节，是数组或者是其他对象，只能确定对象的大类

typeof可以用来检查一个没有声明的变量而不会报错
```javascript
t; // Uncaught ReferenceError: t is not defined

typeof t; // "undefined"
```

## 参考链接

- [阮一峰JavaScript教程-数据类型](https://wangdoc.com/javascript/types/index.html)
- [现代JavaScript教程-数据类型](https://zh.javascript.info/types)
- [Symbol-ES6入门](https://es6.ruanyifeng.com/#docs/symbol)