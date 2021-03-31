# 类继承和原型继承的区别

1. [Object.getPrototypeOf()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/GetPrototypeOf) 结果不同。类继承中子类会`[[Prototype]]`链接到父类，原型继承中的构造函数都是通过`prototype`指向的原型对象相互联系的。一般在原型继承中，子构造函数的原型对象是父构造函数，或者子构造函数的原型对象`[[Prototype]]`链接到父构造函数的原型对象。
2. `this`的创造`顺序`不同。
3. 子类实例的构建，基于父类实例，只有`super`方法才能调用父类实例。

子类必须在`constructor`方法中调用`super`方法，否则新建实例时会报错。这是因为子类自己的`this`对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用`super`方法，子类就得不到`this`对象。

```javascript
// B 继承了父类 A，但是它的构造函数没有调用 super 方法，导致新建实例时报错。
class A {}

class B extends A {
  constructor () {}
}

let c = new B() // ReferenceError 报错
```

[参考：ES6 入门](https://es6.ruanyifeng.com/#docs/class-extends)

### 类声明

函数声明和类声明之间的一个重要区别在于, 函数声明会提升，类声明不会。你首先需要声明你的类，然后再访问它，否则类似以下的代码将抛出 ReferenceError：

```javascript
// 会报错
let p = new Rectangle() // ReferenceError

class Rectangle {}
```

### 用原型和静态方法绑定 this

当调用静态或原型方法时没有指定 this 的值，那么方法内的 this 值将被置为`undefined`。即使你未设置 "use strict" ，因为 class 体内部的代码总是在`严格模式`下执行。

```javascript
class Animal {
  speak () {
    return this
  }
  static eat () {
    return this
  }
}

let obj = new Animal()
obj.speak() // Animal {}
let speak = obj.speak
speak() // undefined

Animal.eat() // class Animal
let eat = Animal.eat
eat() // undefined
```

如果上述代码通过传统的基于函数的语法来实现，那么依据初始的 this 值，在非严格模式下方法调用会发生自动装箱。若初始值是 undefined，this 值会被设为`全局对象`。

```javascript
function Animal () {}

Animal.prototype.speak = function () {
  return this
}

Animal.eat = function () {
  return this
}

let obj = new Animal()
let speak = obj.speak
speak() // global object (Window)

let eat = Animal.eat
eat() // global object (Window)
```

### extends

使用`extends`关键字扩展子类。如果子类中定义了构造函数，必须先调用`super()`才能使用`this`。`super`用于调用对象的父对象上的函数。

```javascript
class Animal {
  constructor (name) {
    this.name = name
  }

  speak () {
    console.log(`${this.name} makes a noise.`)
  }
}

// 子类
class Dog extends Animal {
  constructor (name) {
    super(name) // 调用父类构造函数并传入 name 参数
  }

  speak () {
    console.log(`${this.name} barks.`)
  }
}

let d = new Dog('Mitzie')
d.speak() // Mitzie barks.
```

也可以继承传统的基于函数的类：

```javascript
function Animal (name) {
  this.name = name
}
Animal.prototype.speak = function () {
  console.log(this.name + ' makes a noise.')
}

class Dog extends Animal {
  speak() {
    super.speak() // 用于调用对象的父对象上的函数。
    console.log(this.name + ' barks.')
  }
}

var d = new Dog('Mitzie')
d.speak() // Mitzie makes a noise.  Mitzie barks.
```

类不能继承常规对象（不可构造的）。如果要继承常规对象，可以改用 [Object.setPrototypeOf()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)。

> [MDN 类](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)
