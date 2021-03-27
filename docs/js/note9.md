# async/await 如何通过同步的方式实现异步

async/await 是一个`自执行`的 [Generator](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator) 函数。利用 Generator 函数的特性把异步的代码写成同步的形式。可以理解为是 Generator 的语法糖。

```javascript
const fetch = require('node-fetch')

// 这里的 * 可以看成 async
function* gen () {
  const url = 'https://api.github.com/users/github'

  // 这里的 yield 可以看成 await
  const result = yield fetch(url)
  console.log(result.bio)
}

const g = gen()
const result = g.next()
result.value.then(data => data.json()).then(data => g.next(data))
```

> [参考](https://es6.ruanyifeng.com/#docs/generator-async)
