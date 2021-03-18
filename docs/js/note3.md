# 实现一个 new

```javascript
function _new (fn, ...args) {
  // 创建空对象，并将新对象的 __proto__ 属性指向构造函数的 prototype
  const obj = Object.create(fn.prototype)
  // 执行构造函数，改变构造函数的this指针，指向新创建的对象（新对象也就有了构造函数的所有属性）
  const ret = fn.apply(obj, args)
  return ret instanceof Object ? ret : obj
}
```

在使用`new`创建对象时，会做以下事情：

1. 创建一个空对象
2. 将新对象的原型指向构造函数的原型
3. 执行构造函数，把构造函数的属性添加到新对象中
4. 返回创建的新对象
