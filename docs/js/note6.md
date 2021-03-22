# 如何理解必包

JavaScript 中的函数运行在它们被定义的作用域里，而不是它们被执行的作用域里。

```javascript
const a = 1

function fn () {
  console.log(a)
}

function fn2 () {
  const a = 2
  fn()
}

fn2() // 1
```

```javascript
const fn = function () {
  const a = 1
  return function fn2 () {
    console.log(a)
  }
}

const fn3 = fn()

fn3() // 1
```
