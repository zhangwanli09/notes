# call、apply、bind

call() 方法的`作用`和 apply() 方法`类似`，区别就是`call()`方法接受的是`参数列表`，而`apply()`方法接受的是一个`参数数组`。`bind()`函数会创建一个新的`绑定函数`。

作用都是`改变`函数`执行时`的上下文（改变函数运行时的`this指向`）。

## call

`call()`方法使用一个指定的`this`值和单独给出的一个或多个参数来调用一个函数。参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)。

语法：

```javascript
// thisArg 可选，非严格模式下，指定为 null 或 undefined 时会自动替换为指向全局对象，原始值会被包装。
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
var sData = 1

function display () {
  console.log(this.sData)
}

display.call() // 打印 1。严格模式 'use strict' 下，Cannot read property 'sData' of undefined。
```

### 实现思路

```javascript
Function.prototype.myCall = function (context) {
  // 第一个参数为 this 指向值，如果没有则指向 window
  context = context || window
  // 将调用 myCall 函数的对象添加到 context 的属性中。
  context.fn = this
  // 将后面传入的参数转为数组，取除第一个 this 指向剩下的所有参数
  const args = [...arguments].slice(1)
  // 执行该函数，并将参数传入
  const result = context.fn(...args)
  // 删除该属性
  delete context.fn
  return result
}
```

## apply

`apply()`方法调用一个具有给定`this`值的函数，以及以一个`数组`（或类数组对象）的形式提供的参数。参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)。

语法：

```javascript
// thisArg 必选，在 func 函数运行时使用的 this 值。
// 非严格模式下，指定为 null 或 undefined 时会自动替换为指向全局对象，原始值会被包装。
func.apply(thisArg, [argsArray])
```

返回值：调用有指定this值和参数的函数的结果。

在调用函数时，可以为其指定一个`this`对象。 this 指当前对象，也就是正在调用这个函数的对象。作用和`call()`类似，只是参数有区别。

示例：

```javascript
// 用 apply 将数组各项添加到另一个数组
const arr = [1, 2]
const arr2 = [3, 4]

arr.push.apply(arr, arr2)
console.log(arr) // [1, 2, 3, 4]

// 对于一些需要写循环以便历数组各项的需求，可以用apply完成以避免循环。
// 用 Math.max/Math.min 求数组中的最大/小值。
const numbers = [5, 6, 2, 3, 7]
const max = Math.max.apply(null, numbers)
console.log(max) // 7
const min = Math.min.apply(null, numbers);
console.log(min) // 2
```

## bind

`bind()`方法创建一个`新的函数`，在 bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)。

语法：

```javascript
// thisArg 调用绑定函数时作为 this 参数传递给目标函数的值。 如果使用new运算符构造绑定函数，则忽略该值。
// 如果 bind 函数的参数列表为空，或者thisArg是null或undefined，执行作用域的 this 将被视为新函数的 thisArg。
function.bind(thisArg[, arg1[, arg2[, ...]]])
```

返回值：返回一个原函数的`拷贝`，并拥有指定的 this 值和初始参数。

bind() 函数会创建一个新的`绑定函数`（bound function，BF）。

示例：

```javascript
// 创建绑定函数
this.a = 2 // 在浏览器中，this 指向全局的 "window" 对象

const obj = {
  a: 1,
  getA: function () {
    return this.a
  }
}

obj.getA() // 返回 1

const fn = obj.getA
fn() // 返回 2，因为函数是在全局作用域中调用的

const fn2 = fn.bind(obj)
fn2() // 返回 1，创建一个新函数，把 'this' 绑定到 obj 对象

// ---

// 使一个函数拥有预设的初始参数。当绑定函数被调用时，这些参数会被插入到目标函数的参数列表的开始位置，传递给绑定函数的参数会跟在它们后面。
function sum (a, b) {
  return a + b
}

let sumBind = sum.bind(null, 1)

sumBind(2) // 1 + 2 = 3
sumBind(5, 7) // 1 + 5 = 6，第二个参数7被忽略
```
