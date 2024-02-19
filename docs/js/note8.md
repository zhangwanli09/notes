# 防抖，节流

## 防抖（定时器）

使用`定时器`实现。触发高频事件后 n 秒内函数只会执行一次，如果 n 秒内再次触发，则重新计算时间。使用闭包实现。

```javascript
function debounce (fn, delay) {
  let timer
  return function () {
    clearTimeout(timer)
    timer = setTimeout(() => {
      fn()
    }, delay)
  }
}

// 测试
function fn () {
  console.log(1)
}

const dFn = debounce(fn, 1000)

dFn()
dFn()
dFn()
```

## 节流（变量）

使用`变量`控制。在 n 秒内只会执行一次，

```javascript
function throttle (fn, delay) {
  let trigger
  return function () {
    if (trigger) return
    trigger = true
    fn()
    setTimeout(() => {
      trigger = false
    }, delay)
  }
}

// 测试
function fn () {
  console.log(1)
}

const tFn = throttle(fn, 1000)

tFn()
tFn()
tFn()
```
