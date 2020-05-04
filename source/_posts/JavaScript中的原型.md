---
title: JavaScript中的原型
toc: true
date: 2020-05-03 13:12:37
tags: js
categories:
- 前端
- 基础
---

## 简述
我们创建的每个函数都有一个prototype(原型)属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。
<!--more-->

无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。

在默认情况下，所有原型对象都会自动获得一个constructor属性，这个属性是一个指向prototype属性所在函数的指针。

```javascript
function foo() {};

foo.prototype.constructor === foo; //true
```
![原型简述](http://image.manylu.com/js-prototype-1.png)

## 实例对象与原型对象
### 关系简述
```javascript
function Person() { };

var lucy = new Person();

console.log(lucy.constructor === Person); //true
//在实例对象上找不到的属性和方法会在其构造函数的原型对象上进一步查找
```


![实例对象与原型对象](http://image.manylu.com/js-prototype-2.png)


### 实例对象如何访问原型
Firefox、Safari和Chrome在每个对象上都支持一个属性\_\_proto\_\_；这个连接存在于实例和构造函数的原型对象之间

![对象的__proto__属性](http://image.manylu.com/js-prototype-3.png)

其它方法访问对象的[[Prototype]]属性
- isPrototypeOf()：间接确定对象的原型
```javascript
console.log(Person.prototype.isPrototypeOf(lucy)); //true
```

- Object.getPrototypeOf()：直接返回对象的原型
```javascript
console.log(Object.getPrototypeOf(lucy) === Person.prototype); //true
```

### 属性判断
当为对象实例添加一个属性时，这个属性就会屏蔽原型对象中保存的同名属性，使用hasOwnProperty()方法可以检测一个属性是存在实例对象中，还是存在于原型中。
```javascript
function Person() {
	name: "Bob"
};

var person = new Person();
console.log(person.hasOwnProperty("name")); //false

person.name = "Lucy";
console.log(person.hasOwnProperty("name")); //true

```

### 原型的动态性
由于在原型中查找值的过程是一次搜索，因此我们对原型对象所做的任何修改都能够立即从实例上反映出来，即使是先创建了实例后修改原型也照样如此。
```javascript
function Person() {};

var person = new Person();

Person.prototype.sayHi = function() {
	alert("Hi");
};

person.sayHi(); // "Hi" 
```

### 重写原型
尽管可以随时为原型添加属性和方法，并且修改能够立即在所有对象实例中反映出来，但如果是重写整个原型对象，情况就有所不同。

调用构造函数时会为实例添加一个指向最初原型的[[Prototype]]指针，但是把原型修改为另一个对象就等于切断了构造函数与最初原型之间的联系。

```javascript
function Person() {};

var person = new Person();

Person.prototype = {
	constructor: Person,
	name: "Lucy"
};

console.log(person.name); // undefined
```
![重写原型对象之前](http://image.manylu.com/js-prototype-4.png)
![重写原型对象之后](http://image.manylu.com/js-prototype-5.png)

重写原型对象切断了现有原型与任何之前已经存在的对象实例之间的联系。对象实例引用的仍然是最初的原型。

## 原型模式缺点
原型中所有属性是被很多实例所共享的，这种共享对于函数来说非常合适，对于包含基本值的属性值也还说得过去(因为在实例对象上添加一个同名属性会屏蔽掉原型中的对应属性)，但是对于包含引用类型值的属性来说问题比较突出，因为任何一个实例对象修改了原型中的引用类型的属性值后，所有其他实例对象中对应的引用类型值均发生改变。

```javascript
function Person() {};

Person.prototype = {
	constructor: Person,
	friends: ["Handy", "Mona"]
};

var person1 = new Person();
var person2 = new Person();

person1.friends.push("May");
console.log(person1.friends); //["Handy", "Mona","May"]
console.log(person2.friends); //["Handy", "Mona","May"]
console.log(person1.friends === person2.friends); //true
```

所以自定义类型的最常见方式就是组合使用构造函数模式和原型模式。构造函数模式用于定义实例属性，原型模式用于定义方法和共享的属性。

## 原型链
简单回顾一下构造函数、原型和实例的关系：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针(constructor),而实例都包含一个指向原型对象的内部指针[[Prototype]]。

那么，假如，我们让原型对象等于另一个类型的实例，则此时的原型对象将包含指向另一个原型的指针[[Prototype]],它也有自己的原型，因此会形成一个原型链(prototype chain)：对象到原型，再到原型的原型...如果一层层上溯，所有对象的原型最终都可以上溯到Object.prototype,也就是说，所有对象都继承了Object.prototype的属性。
```javascript
function Animal() {
	this.eats = true;
};

Animal.prototype.walk = function() {
	return "Animal's walk";
};

function Pig() {
	this.earNum = 2;
};

Pig.prototype = new Animal(); //继承了Animal

Pig.prototype.sleep = function() {
	return "Pig's sleep";
};

function FlowerPig() {
	this.skin = "flower";
}

FlowerPig.prototype = new Pig(); //继承了Pig

var pig1 = new FlowerPig();

console.log(pig1.skin); //flower
console.log(pig1.sleep()); // "Pig's sleep"
console.log(pig1.earNum); //2
console.log(pig1.walk());// "Animal's walk"
console.log(pig1.eats); //true
```

![代码所示原型链](http://image.manylu.com/js-prototype-6.png)

当以读取模式访问一个实例属性时，首先会在实例中搜索该属性。如果没有找到该属性，则会继续搜索实例的原型。在通过原型链实现继承的情况下，搜索过程就得以沿着原型链继续向上，搜索过程总是要一环一环的前行到原型链末端才会停下来。

在通过原型来实现继承时，原型实际上是另一个类型的实例，所以原先的实例属性也就变成了现在的原型属性，所以还是需要注意包含引用类型值的原型。

## 参考内容
- 《JavaScript高级程序设计第三版》 第六章
- [阮一峰JavaScript教程-对象的继承](https://wangdoc.com/javascript/oop/prototype.html)