# 类继承和原型继承的区别

函数声明和类声明之间的一个重要区别在于, 函数声明会提升，类声明不会。你首先需要声明你的类，然后再访问它，否则类似以下的代码将抛出 ReferenceError：

```javascript
// 会报错
let p = new Rectangle() // ReferenceError

class Rectangle {}
```

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
