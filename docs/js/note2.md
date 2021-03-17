# call、apply、bind

call() 方法的`作用`和 apply() 方法`类似`，区别就是`call()`方法接受的是`参数列表`，而`apply()`方法接受的是一个`参数数组`。

### call

`call()`方法使用一个指定的`this`值和单独给出的一个或多个参数来调用一个函数。参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)。

语法：

```javascript
function.call(thisArg, arg1, arg2, ...)
```

返回值：使用调用者提供的 this 值和参数调用该函数的返回值。若该方法没有返回值，则返回 undefined。

可以使用`call()`来实现`继承`：写一个方法，然后让另外一个新的对象来继承它（而不是在新对象中再写一次这个方法）。

示例：

```javascript
// 实现继承
function Product (name, price) {
  this.name = name
  this.price = price
}

function Food (name, price) {
  Product.call(this, name, price)
  this.category = 'food'
}

console.log(new Food('cheese', 5).name) // cheese


// 使用 call 方法调用函数并指定上下文的 this
function fn () {
  console.log(this.a)
}

const obj = { a: 1 }

fn.call(obj) // 打印 1


// 不指定第一个参数，在严格模式下，this 的值将会是 undefined。
const sData = 1

function display () {
  console.log(this.sData)
}

display.call() // 打印 1。严格模式 'use strict' 打印 undefined。
```

### apply

