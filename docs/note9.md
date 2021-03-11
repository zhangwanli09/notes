# Vue响应式原理中Object.defineProperty有什么缺陷？为什么Vue3.0采用Proxy?

Object.defineProperty和Proxy，都可以用来做数据劫持，典型应用如数据的双向绑定。

### Object.defineProperty的缺陷

语法：
```javascript
Object.defineProperty(obj, prop, descriptor)
```

1. 不能监听数组变化，需要重写数组实现响应式。
2. 必须遍历对象的每个属性，如果嵌套对象，还要逐层遍历，直到把每个对象的每个属性都调用Object.defineProperty()为止。（源码 walk方法）。
3. Object.defineProperty()要配合Object.keys()使用，多了层嵌套。

源码：
```javascript
/**
 * Walk through all properties and convert them into
 * getter/setters. This method should only be called when
 * value type is Object.
 */
Observer.prototype.walk = function walk (obj) {
  var keys = Object.keys(obj);
  for (var i = 0; i < keys.length; i++) {
    defineReactive$$1(obj, keys[i]);
  }
};
```

### Vue重写数组

解决Object.defineProperty()无法监听数组变化问题。

源码：
```javascript
/*
 * not type checking this file because flow doesn't play well with
 * dynamically accessing methods on Array prototype
 */

var arrayProto = Array.prototype;
var arrayMethods = Object.create(arrayProto);

var methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
];

/**
 * Intercept mutating methods and emit events
 */
methodsToPatch.forEach(function (method) {
  // cache original method
  var original = arrayProto[method];
  def(arrayMethods, method, function mutator () {
    var args = [], len = arguments.length;
    while ( len-- ) args[ len ] = arguments[ len ];

    var result = original.apply(this, args);
    var ob = this.__ob__;
    var inserted;
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args;
        break
      case 'splice':
        inserted = args.slice(2);
        break
    }
    if (inserted) { ob.observeArray(inserted); }
    // notify change
    ob.dep.notify();
    return result
  });
});
```

1. 源码中使用Object.create()的目的是避免全局污染Array数组，只影响Vue里的数组。
2. vue对push，pop，splice等方法进行了hack，如果加入新对象，对新对象进行响应式化。
3. 大概意思是调用数组的原型方法，如果推入了一个对象， 就调用observe方法将加入的对象改成响应式对象，然后再通知观察者进行更新。

Vue中默认的做法就是在数组实例与它的原型之间，插入了一个新的原型对象，这个原型方法实现了这些变异方法，也就拦截了真正数组原型上的方法（因为原型链的机制，找到了就不会继续往上找了）。
变异方法中增加了自定义逻辑，也调用了真正数组原型上的方法，即实现了目的，也不会对正常使用造成影响。

### 存在的问题

Object.defineProperty的缺陷导致如果直接改变数组下标就无法hack，所有Vue提供$set方法解决此问题。

### Vue检测变化的注意事项

1. 对于对象，Vue无法检测property的添加或删除。由于Vue会在初始化实例时对property执行getter/setter转化，所以property必须在data对象上存在才能让Vue将它转换为响应式。
2. 对于数组，Vue不能检测以下数组的变动：
    1. 当利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`。
    2. 当修改数组长度时，例如：`vm.items.length = newLength`。

解决方案：
```javascript
// 解决对象问题
// 向嵌套对象添加响应式属性
Vue.set(object, propertyName, value)

// 解决数组问题
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)

// 解决数组第二类问题，可以使用 splice
vm.items.splice(newLength)
```

### Proxy

Proxy对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。

语法：
```javascript
const p = new Proxy(target, handler)
```

特点：

1. 无操作转发代理。代理会将所有应用到它的操作转发到这个对象上。
    ```javascript
    let target = {}
    let p = new Proxy(target, {})
    p.a = 'Hi' // 操作转发到目标
    console.log(target.a) // Hi 操作已经被正确地转发
    ```
2. 针对整个对象。无论对象内部有多少key，都可以触发set等方法。（不需要用Object.keys()遍历）。
3. 支持数组，不需要对数组方法进行重写，省去很多hack。
4. Proxy也不支持嵌套，也需要逐层遍历来解决，不过它的写法是在get里递归调用Proxy并返回。

### 总结

Object.defineProperty和Proxy区别
1. 当使用defineProperty，修改原来的obj对象就可以触发拦截，而使用Proxy，就必须修改代理对象（即Proxy的实例才可以触发拦截）。
2. defineProperty必须深层遍历嵌套的对象。Proxy不需要对数组方法进行重写。

Proxy对比Object.defineProperty优势
1. Proxy的handler参数有13种拦截方法，比Object.defineProperty丰富。
2. Proxy是新标准，性能优于Object.defineProperty。
