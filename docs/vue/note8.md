# Vue响应式原理

### 基于数据劫持的优势

1. 无需显示调用。例如Vue运用数据劫持+发布订阅，在数据变动时发布消息给订阅者，触发相应的监听回调驱动视图更新。
2. 可精确得知变化数据。例如劫持属性的setter，当属性改变可以得到变化的内容，不需要做额外的diff操作。

Vue官网对响应式原理的描述：

每个组件实例都对应一个watcher实例，它会在组件渲染过程中把接触过的数据property记录为依赖。之后当依赖项的setter触发时，会通知watcher，从而使它关联的组件重新渲染。

![响应式原理图](../imgs/img1.png ':size=600')

### 响应式对象

Vue实现响应式的核心是利用`Object.defineProperty()`，具体用法参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)。

当把对象传入Vue实例作为data选项，Vue将遍历此对象所有的属性，并使用Object.defineProperty把这些属性全部转为`getter/setter`。可以简单的把这种对象称为响应式对象。Object.defineProperty是ES5中一个无法shim的特性，这也就是Vue不支持IE8以及更低版本浏览器的原因。

#### initState

在Vue的初始化阶段，`_init`方法执行的时候，会执行`initState(vm)`方法，它主要是对props、methods、data、computed和watch等属性做初始化操作。

```javascript
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

重点分析props和data的初始化：

`initProps`的主要过程是遍历定义的props。遍历过程主要做两件事：
1. 调用`defineReactive`方法，把每个props的值变成响应式。可以通过`vm._props.xxx`访问到定义props中对应的属性。
2. 通过`proxy`把`vm._props.xxx`的访问代理到`vm.xxx`上。

```javascript
function initProps (vm: Component, propsOptions: Object) {
  const propsData = vm.$options.propsData || {}
  const props = vm._props = {}
  // cache prop keys so that future props updates can iterate using Array
  // instead of dynamic object key enumeration.
  const keys = vm.$options._propKeys = []
  const isRoot = !vm.$parent
  // root instance props should be converted
  if (!isRoot) {
    toggleObserving(false)
  }
  for (const key in propsOptions) {
    keys.push(key)
    const value = validateProp(key, propsOptions, propsData, vm)
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      const hyphenatedKey = hyphenate(key)
      if (isReservedAttribute(hyphenatedKey) ||
          config.isReservedAttr(hyphenatedKey)) {
        warn(
          `"${hyphenatedKey}" is a reserved attribute and cannot be used as component prop.`,
          vm
        )
      }
      defineReactive(props, key, value, () => {
        if (!isRoot && !isUpdatingChildComponent) {
          warn(
            `Avoid mutating a prop directly since the value will be ` +
            `overwritten whenever the parent component re-renders. ` +
            `Instead, use a data or computed property based on the prop's ` +
            `value. Prop being mutated: "${key}"`,
            vm
          )
        }
      })
    } else {
      defineReactive(props, key, value)
    }
    // static props are already proxied on the component's prototype
    // during Vue.extend(). We only need to proxy props defined at
    // instantiation here.
    if (!(key in vm)) {
      proxy(vm, `_props`, key)
    }
  }
  toggleObserving(true)
}
```

`initData`的主要过程也是做两件事情：
1. 对data函数返回对象遍历，通过`proxy`把每一个值`vm._data.xxx`都代理到`vm.xxx`上。
2. 调用`observe`方法观测整个data的变化，把data变成响应式，可以通过`vm._data.xxx`访问到定义data返回函数中对应的属性。

```javascript
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    )
  }
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  while (i--) {
    const key = keys[i]
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(
          `Method "${key}" has already been defined as a data property.`,
          vm
        )
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${key}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    } else if (!isReserved(key)) {
      proxy(vm, `_data`, key)
    }
  }
  // observe data
  observe(data, true /* asRootData */)
}
```

`props`和`data`的初始化都是把它们变成响应式对象。

#### proxy

`proxy`的作用是把`props`和`data`上的属性代理到`vm`实例上。所以定义在props中的属性，也可以通过vm实例访问到。

```javascript
const sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
}

export function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

通过Object.defineProperty把`target[sourceKey][key]`的读写变成对`target[key]`的读写。
1. 对于props，对`vm._props.xxx`的读写变成了`vm.xxx`的读写。
2. 对于data，对`vm._data.xxx`的读写变成了对`vm.xxx`的读写。

#### observe

observe是用来监听数据变化的。

observe方法作用是给非VNode的对象类型数据添加一个Observer，如果已经添加过则直接返回，否则在满足一定条件下去实例化一个Observer对象实例。

observe源码：
```javascript
/**
 * Attempt to create an observer instance for a value,
 * returns the new observer if successfully observed,
 * or the existing observer if the value already has one.
 */
export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
```

#### Observer

Observer作用是给对象属性添加getter和setter，用于依赖收集和派发更新。

Observer的构造函数逻辑：
1. 首先实例化Dep对象。
2. 接着通过执行def函数把自身实例添加到数据对象value的__ob__属性上。def函数是一个非常简单的Object.defineProperty封装。
3. 对value做判断，对于数组会调用protoAugment或copyAugment，observeArray方法，纯对象则调用walk方法。
    1. 用hasProto判断浏览器是否支持__proto__，如果支持则使用protoAugment函数来覆盖原型，否则调用copyAugment函数将拦截器中的方法挂载到value上。（这里涉及到Vue对部分数组方法的重写）
    2. observeArray是遍历数组再次调用observe方法。
    3. walk是遍历对象的key调用defineReactive方法。

def源码：
```javascript
/**
 * Define a property.
 */
export function def (obj: Object, key: string, val: any, enumerable?: boolean) {
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: !!enumerable,
    writable: true,
    configurable: true
  })
}
```

Observer源码：
```javascript
/**
 * Observer class that is attached to each observed
 * object. Once attached, the observer converts the target
 * object's property keys into getter/setters that
 * collect dependencies and dispatch updates.
 */
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that have this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      if (hasProto) {
        protoAugment(value, arrayMethods)
      } else {
        copyAugment(value, arrayMethods, arrayKeys)
      }
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  /**
   * Walk through all properties and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  /**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```

关于Vue对数组方法重写的部分源码：

```javascript
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)

const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]

/**
 * Intercept mutating methods and emit events
 */
methodsToPatch.forEach(function (method) {
  // cache original method
  const original = arrayProto[method]
  def(arrayMethods, method, function mutator (...args) {
    const result = original.apply(this, args)
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // notify change
    ob.dep.notify()
    return result
  })
})
```

#### defineReactive

defineReactive的作用是定义一个响应式对象，给对象添加getter和setter。

1. 初始化Dep对象的实例。
2. 拿到obj的属性描述符，然后对子对象递归调用observe方法，保证子属性也能够变成响应式对象。
3. 最后利用Object.defineProperty给obj的属性添加getter和setter。

源码：
```javascript
/**
 * Define a reactive property on an Object.
 */
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }

  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) return
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```

#### 总结

响应式对象的核心就是利用`Object.defineProperty`给数据添加了`getter/setter`，目的就是为了在访问数据和修改数据时能自动执行一些逻辑：getter做依赖收集，setter做派发更新。

### 依赖收集

响应式对象的getter部分就是做依赖收集的。defineReactive方法中重点关注：

1. `const dep = new Dep()`实例化一个Dep实例。
2. get函数中通过`dep.depend()`做依赖收集。这里涉及到`childOb`的判断逻辑。

#### Dep

Dep是整个getter依赖收集的核心。

Dep源码：
```javascript
let uid = 0

/**
 * A dep is an observable that can have multiple
 * directives subscribing to it.
 */
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
      subs.sort((a, b) => a.id - b.id)
    }
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

// The current target watcher being evaluated.
// This is globally unique because only one watcher
// can be evaluated at a time.
Dep.target = null
const targetStack = []

export function pushTarget (target: ?Watcher) {
  targetStack.push(target)
  Dep.target = target
}

export function popTarget () {
  targetStack.pop()
  Dep.target = targetStack[targetStack.length - 1]
}
```

Dep实际上是对Watcher的管理。注意点：

1. 静态属性`target`，这是全局唯一`Watcher`。在同一时间只能有一个全局`Watcher`被计算。
2. 它的自身属性`subs`也是`Watcher`的数组。

#### Watcher

Watcher的构造函数中定义了一些和Dep相关的属性：

```javascript
this.deps = []
this.newDeps = []
this.depIds = new Set()
this.newDepIds = new Set()
```
其中，this.deps和this.newDeps表示Watcher实例持有的Dep实例的数组；而this.depIds和this.newDepIds分别代表this.deps和this.newDeps的id Set（Set是ES6的数据类型）。

还定义了些原型方法，依赖收集相关的有`get`、`addDep`、`cleanupDeps`方法。

源码：

```javascript
let uid = 0

/**
 * A watcher parses an expression, collects dependencies,
 * and fires callback when the expression value changes.
 * This is used for both the $watch() api and directives.
 */
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = noop
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  /**
   * Add a dependency to this directive.
   */
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }

  /**
   * Clean up for dependency collection.
   */
  cleanupDeps () {
    let i = this.deps.length
    while (i--) {
      const dep = this.deps[i]
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this)
      }
    }
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }

  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }

  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
  run () {
    if (this.active) {
      const value = this.get()
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        this.value = value
        if (this.user) {
          try {
            this.cb.call(this.vm, value, oldValue)
          } catch (e) {
            handleError(e, this.vm, `callback for watcher "${this.expression}"`)
          }
        } else {
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }

  /**
   * Evaluate the value of the watcher.
   * This only gets called for lazy watchers.
   */
  evaluate () {
    this.value = this.get()
    this.dirty = false
  }

  /**
   * Depend on all deps collected by this watcher.
   */
  depend () {
    let i = this.deps.length
    while (i--) {
      this.deps[i].depend()
    }
  }

  /**
   * Remove self from all dependencies' subscriber list.
   */
  teardown () {
    if (this.active) {
      // remove self from vm's watcher list
      // this is a somewhat expensive operation so we skip it
      // if the vm is being destroyed.
      if (!this.vm._isBeingDestroyed) {
        remove(this.vm._watchers, this)
      }
      let i = this.deps.length
      while (i--) {
        this.deps[i].removeSub(this)
      }
      this.active = false
    }
  }
}
```

#### 过程分析

依赖收集大致过程：
1. Vue的`mount`过程中实例化一个渲染`watcher`。
2. 执行`pushTarget(this)`，把当前的渲染watcher赋值给`Dep.target`。
3. 执行`vm._render()`（通过参数`mountComponent`传入）生成vnode的过程触发数据的`getter`，内部调用`dep.depend()`，把当前的`watcher`订阅到这个数据持有的dep的`subs`中，为后续数据变化通知做准备。

Vue的mount的过程是通过`mountComponent`函数，其中有段重要逻辑：

```javascript
// mountComponent函数部分代码

updateComponent = () => {
  vm._update(vm._render(), hydrating)
}

new Watcher(vm, updateComponent, noop, {
  before () {
    if (vm._isMounted && !vm._isDestroyed) {
      callHook(vm, 'beforeUpdate')
    }
  }
}, true /* isRenderWatcher */)
```

当实例化一个渲染`watcher`的时候（参考上面Watcher源码）：

首先进入`Watcher`的构造函数，执行`this.get()`方法。进入get函数，首先会执行：
```javascript
pushTarget(this)

// dep.js
export function pushTarget (target: ?Watcher) {
  targetStack.push(target)
  Dep.target = target
}
```
把当前渲染的`watcher`赋值给`Dep.target`，并压栈（为了恢复用）。接着执行：
```javascript
value = this.getter.call(vm, vm)
```
this.getter对应的是Watcher传入的第二个参数`updateComponent`函数，实际上是在执行：
```javascript
vm._update(vm._render(), hydrating)
```
它会先执行`vm._render()`方法生成渲染VNode，在这个过程中对vm上的数据访问，这时就会触发数据对象的getter。（注意：在Vue的初始化阶段，`initState(vm)`方法中`initData(vm)`，内部会调用`observe`方法把data变成响应式）。

每个对象值的getter都有一个dep，在触发getter时会调用`dep.depend()`，也就会执行`Dep.target.addDep(this)`。
```javascript
// dep.js
depend () {
  if (Dep.target) {
    Dep.target.addDep(this)
  }
}
```

上面有提到，因为`Dep.target`已经被赋值为当前`watcher`，那么就执行`addDep`方法：
```javascript
/**
 * Add a dependency to this directive.
 */
addDep (dep: Dep) {
  const id = dep.id
  if (!this.newDepIds.has(id)) {
    this.newDepIds.add(id)
    this.newDeps.push(dep)
    if (!this.depIds.has(id)) {
      dep.addSub(this)
    }
  }
}
```
这里会做些逻辑判断，保证统一数据不会被添加多次，然后执行`dep.addSub(this)`，内部会执行`this.subs.push(sub)`，把当前的`watcher`订阅到这个数据持有的dep的`subs`中，目的是为了后续数据变化时能通知到哪些subs做准备。
```javascript
addSub (sub: Watcher) {
  this.subs.push(sub)
}
```

所以在`vm._render()`过程中，会触发所有数据的getter，完成依赖收集。

完成依赖收集后的逻辑：

```javascript
if (this.deep) {
  traverse(value)
}
popTarget()
this.cleanupDeps()
```

1. `traverse(value)`递归访问value，触发它所有子项的getter。
2. `popTarget()`把Dep.target恢复到上一个状态，因为当前vm的数据依赖收集已经完成，那么对应的渲染Dep.target也需要改变。
  ```javascript
  export function popTarget () {
    targetStack.pop()
    Dep.target = targetStack[targetStack.length - 1]
  }
  ```
3. `this.cleanupDeps()`依赖清空的过程。
  ```javascript
  /**
   * Clean up for dependency collection.
   */
  cleanupDeps () {
    let i = this.deps.length
    while (i--) {
      const dep = this.deps[i]
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this)
      }
    }
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }
  ```

考虑到Vue是数据驱动的，所以每次数据变化都会重新render，那么vm._render()方法又会再次执行，并再次触发数据的getters，所以Watcher在构造函数中会初始化2个Dep实例数组，newDeps表示新添加的Dep实例数组，而deps表示上一次添加的Dep实例数组。

在执行`cleanupDeps`函数的时候，会首先遍历deps，移除对dep.subs数组中Wathcer的订阅，然后把newDepIds和depIds交换，newDeps和deps交换，并把newDepIds和newDeps清空。

那么为什么需要做deps订阅的移除呢，在添加deps的订阅过程，已经能通过id去重避免重复订阅了。

考虑到一种场景，我们的模板会根据v-if去渲染不同子模板a和b，当我们满足某种条件的时候渲染a的时候，会访问到a中的数据，这时候我们对a使用的数据添加了getter，做了依赖收集，那么当我们去修改a的数据的时候，理应通知到这些订阅者。那么如果我们一旦改变了条件渲染了b模板，又会对b使用的数据添加了getter，如果我们没有依赖移除的过程，那么这时候我去修改a模板的数据，会通知a数据的订阅的回调，这显然是有浪费的。

因此Vue设计了在每次添加完新的订阅，会移除掉旧的订阅，这样就保证了在我们刚才的场景中，如果渲染b模板的时候去修改a模板的数据，a数据订阅回调已经被移除了，所以不会有任何浪费。

#### 总结

收集依赖的目的是为了当这些响应式数据发生变化，触发它们的setter的时候，能知道应该通知哪些订阅者去做相应的逻辑处理，这个过程叫派发更新，其实Watcher和Dep就是一个非常经典的观察者设计模式的实现。

### 派发更新

上面依赖收集的目的是为了当修改数据时，可以对相关的依赖派发更新。参考`setter`部分逻辑：

```javascript
// defineReactive 方法内 setter 部分
Object.defineProperty(obj, key, {
  enumerable: true,
  configurable: true,
  // ...省略get
  set: function reactiveSetter (newVal) {
    const value = getter ? getter.call(obj) : val
    /* eslint-disable no-self-compare */
    if (newVal === value || (newVal !== newVal && value !== value)) {
      return
    }
    /* eslint-enable no-self-compare */
    if (process.env.NODE_ENV !== 'production' && customSetter) {
      customSetter()
    }
    // #7981: for accessor properties without setter
    if (getter && !setter) return
    if (setter) {
      setter.call(obj, newVal)
    } else {
      val = newVal
    }
    childOb = !shallow && observe(newVal)
    dep.notify()
  }
})
```

setter中2个关键逻辑：
1. shallow如果是false，会对新设置的值变成响应式对象`childOb = !shallow && observe(newVal)`。
2. 通知所有的订阅者`dep.notify()`。

#### 过程分析

当对响应式数据做修改，会触发setter逻辑，会调用`dep.notify()`，它是Dep的实例方法。

```javascript
export default class Dep {
  // ...省略
  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
      subs.sort((a, b) => a.id - b.id)
    }
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
```

遍历所有`subs`，也就是Watcher的实例数组，然后调用每个`Watcher`的`update`方法。

```javascript
export default class Watcher {
  // ...省略

  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }
}
```

对于Watcher的不同状态，会执行不同的逻辑，在一般组件数据更新的场景，会走最后一个逻辑`queueWatcher(this)`。

#### queueWatcher

```javascript
const queue: Array<Watcher> = []
let has: { [key: number]: ?true } = {}
let waiting = false
let flushing = false
let index = 0

/**
 * Push a watcher into the watcher queue.
 * Jobs with duplicate IDs will be skipped unless it's
 * pushed when the queue is being flushed.
 */
export function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      waiting = true

      if (process.env.NODE_ENV !== 'production' && !config.async) {
        flushSchedulerQueue()
        return
      }
      nextTick(flushSchedulerQueue)
    }
  }
}
```
这里引入一个队列的概念，也是Vue在做派发更新时的优化点，它并不会每次数据改变都会触发watcher的回调，而是把这些watcher先添加到队列`queue`里，然后在`nextTick`后执行`flushSchedulerQueue`。

几个细节点：
1. has保证同一个Watcher只添加一次。
2. waiting保证nextTick只会被调用一次。
3. nextTick可以理解它是在下一个tick，也就是异步的去执行flushSchedulerQueue。

#### flushSchedulerQueue

flushSchedulerQueue的一些重要逻辑：
1. 队列排序。对队列做从小到大排序，目的是确保以下几点：
    1. 组件的更新由父到子，因为父组件的创建过程是先于子的，所以watcher的创建也是先父后子，执行顺序也应该保持先父后子。
    2. 用户的自定义watcher要优先于渲染watcher执行，因为用户自定义watcher是在渲染watcher之前创建的。
    3. 如果一个组件在父组件的watcher执行期间被销毁，那么它对应的watcher执行都可以被跳过，所以父组件的watcher应该先执行。
2. 队列遍历。拿到对应的watcher，执行`watcher.run()`。
3. 状态恢复。执行`resetSchedulerState`函数。它的逻辑就是把这些控制流程状态的一些变量恢复到初始值，把watcher队列清空。

```javascript
const queue: Array<Watcher> = []
const activatedChildren: Array<Component> = []
let has: { [key: number]: ?true } = {}
let circular: { [key: number]: number } = {}
let flushing = false
let index = 0

/**
 * Flush both queues and run the watchers.
 */
function flushSchedulerQueue () {
  currentFlushTimestamp = getNow()
  flushing = true
  let watcher, id

  // Sort queue before flush.
  // This ensures that:
  // 1. Components are updated from parent to child. (because parent is always
  //    created before the child)
  // 2. A component's user watchers are run before its render watcher (because
  //    user watchers are created before the render watcher)
  // 3. If a component is destroyed during a parent component's watcher run,
  //    its watchers can be skipped.
  queue.sort((a, b) => a.id - b.id)

  // do not cache length because more watchers might be pushed
  // as we run existing watchers
  for (index = 0; index < queue.length; index++) {
    watcher = queue[index]
    if (watcher.before) {
      watcher.before()
    }
    id = watcher.id
    has[id] = null
    watcher.run()
    // in dev build, check and stop circular updates.
    if (process.env.NODE_ENV !== 'production' && has[id] != null) {
      circular[id] = (circular[id] || 0) + 1
      if (circular[id] > MAX_UPDATE_COUNT) {
        warn(
          'You may have an infinite update loop ' + (
            watcher.user
              ? `in watcher with expression "${watcher.expression}"`
              : `in a component render function.`
          ),
          watcher.vm
        )
        break
      }
    }
  }

  // keep copies of post queues before resetting state
  const activatedQueue = activatedChildren.slice()
  const updatedQueue = queue.slice()

  resetSchedulerState()

  // call component updated and activated hooks
  callActivatedHooks(activatedQueue)
  callUpdatedHooks(updatedQueue)

  // devtool hook
  /* istanbul ignore if */
  if (devtools && config.devtools) {
    devtools.emit('flush')
  }
}
```

#### watcher.run()

watcher.run()逻辑分析：

```javascript
export default class Watcher {
  // ...省略

  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
  run () {
    if (this.active) {
      const value = this.get()
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        this.value = value
        if (this.user) {
          try {
            this.cb.call(this.vm, value, oldValue)
          } catch (e) {
            handleError(e, this.vm, `callback for watcher "${this.expression}"`)
          }
        } else {
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }
}
```

1. 通过执行`this.get()`得到当前的值，判断如果满足新旧值不等、新值是对象类型、deep模式任何一个条件，则执行watcher的回调`cb`，回调函数会传入新值和旧值，这就是当添加自定义watcher的时候能在回调函数的参数中拿到新旧值的原因。
2. 对于渲染watcher而言，在执行this.get()方法求值的时候，会执行getter方法：
  ```javascript
  updateComponent = () => {
    vm._update(vm._render(), hydrating)
  }
  ```
  这就是当修改组件相关的响应式数据的时，会触发组件重新渲染的原因。

#### 总结

派发更新实际上就是当数据发生变化时，触发setter逻辑，把在依赖收集过程中订阅的所有观察者（watcher），都触发它们的update过程，这个过程利用了队列做了优化，在nextTick后执行所有watcher的run，最后执行它们的回调函数。

### 原理图

![原理图](../imgs/img2.png ':size=600')

> 参考资料：
>
> https://ustbhuangyi.github.io/vue-analysis/v2/reactive/
