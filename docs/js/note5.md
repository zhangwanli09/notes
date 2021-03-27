# 柯里化（Currying）

柯里化（Currying）是一种函数的`转换`，将函数从`fn(a, b, c)`转换为`fn(a)(b)(c)`。把接收多个参数的函数变换为接收一个单一参数（最初函数的第一个参数）的函数，并返回接收剩余参数而且返回结果的新函数的技术。

只传递给函数`一部分参数`来调用它，让它返回一个函数去处理`剩下的参数`。

使用场景：

1. `参数复用`，当在多次调用同一个函数，并且传递的参数绝大多数是相同的，那么该函数可能是一个很好的柯里化候选。
2. `延迟执行`，即如果调用柯里化函数传入参数是不调用的，会将参数添加到数组中存储，等到没有参数传入的时候进行调用。

示例：

```javascript
function curryingAdd () {
  let args = [].slice.call(arguments, 0)
  
  function add () {
    args = [...args, [].slice.call(arguments, 0)]
    return add
  }

  add.toString = function () {
    return args.reduce((t, a) => t + +a, 0)
  }

  return add
}

curryingAdd(1, 2, 3) // 6
curryingAdd(1)(2)(3) // 6
curryingAdd(2, 6)(1) // 9
```

> 参考：
>
> [arguments](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/arguments)
>
> [参考](https://juejin.cn/post/6844903603266650125)