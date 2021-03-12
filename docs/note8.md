# Vue实现双向绑定的原理

### 基于数据劫持的优势

1. 无需显示调用。例如Vue运用数据劫持+发布订阅，在数据变动时发布消息给订阅者，触发相应的监听回调驱动视图更新。
2. 可精确得知变化数据。例如劫持属性的setter，当属性改变可以得到变化的内容，不需要做额外的diff操作。


### 实现思路

1. 通过Object.defineProperty生成监听器Observer监听对象属性，在属性发生变化后通知订阅者。
2. 通过Compile解析编译模版指令，根据指令模版替换数据，绑定相应的更新函数。
3. 通过Watcher衔接Observer和Compile，订阅并收到每个属性变化的通知，执行指令绑定的回调函数更新视图。

每个组件实例都对应一个watcher实例，它会在组件渲染过程中把接触过的数据property记录为依赖。之后当依赖项的setter触发时，会通知watcher，从而使它关联的组件重新渲染。

![响应式原理图](./imgs/img1.png)

### observe

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

### Observer

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
1. Vue的`mount`过程中实例化一个`Watcher`。
2. 执行`pushTarget(this)`，把当前的watcher赋值给`Dep.target`。
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

### Virtual DOM（虚拟DOM）

Vue通过vm._render方法（最终是通过执行createElement方法返回vnode）把实例渲染成一个虚拟Node。

Virtual DOM是使用js对象描述DOM节点。

源码：
```javascript
export default class VNode {
  tag: string | void;
  data: VNodeData | void;
  children: ?Array<VNode>;
  text: string | void;
  elm: Node | void;
  ns: string | void;
  context: Component | void; // rendered in this component's scope
  key: string | number | void;
  componentOptions: VNodeComponentOptions | void;
  componentInstance: Component | void; // component instance
  parent: VNode | void; // component placeholder node

  // strictly internal
  raw: boolean; // contains raw HTML? (server only)
  isStatic: boolean; // hoisted static node
  isRootInsert: boolean; // necessary for enter transition check
  isComment: boolean; // empty comment placeholder?
  isCloned: boolean; // is a cloned node?
  isOnce: boolean; // is a v-once node?
  asyncFactory: Function | void; // async component factory function
  asyncMeta: Object | void;
  isAsyncPlaceholder: boolean;
  ssrContext: Object | void;
  fnContext: Component | void; // real context vm for functional nodes
  fnOptions: ?ComponentOptions; // for SSR caching
  devtoolsMeta: ?Object; // used to store functional render context for devtools
  fnScopeId: ?string; // functional scope id support

  constructor (
    tag?: string,
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions,
    asyncFactory?: Function
  ) {
    this.tag = tag
    this.data = data
    this.children = children
    this.text = text
    this.elm = elm
    this.ns = undefined
    this.context = context
    this.fnContext = undefined
    this.fnOptions = undefined
    this.fnScopeId = undefined
    this.key = data && data.key
    this.componentOptions = componentOptions
    this.componentInstance = undefined
    this.parent = undefined
    this.raw = false
    this.isStatic = false
    this.isRootInsert = true
    this.isComment = false
    this.isCloned = false
    this.isOnce = false
    this.asyncFactory = asyncFactory
    this.asyncMeta = undefined
    this.isAsyncPlaceholder = false
  }

  // DEPRECATED: alias for componentInstance for backwards compat.
  /* istanbul ignore next */
  get child (): Component | void {
    return this.componentInstance
  }
}
```

VNode是对真实DOM的一种抽象描述，核心的关键属性有标签名，数据，子节点，键值等。由于VNode只是用来映射到真实DOM的渲染，不包含操作DOM的方法，因此非常轻量和简单。

映射到真实的DOM要经历VNode的create、diff、patch等过程。Vue利用createElement方法创建的VNode。

源码：

```javascript
export function createElement (
  context: Component,
  tag: any,
  data: any,
  children: any,
  normalizationType: any,
  alwaysNormalize: boolean
): VNode | Array<VNode> {
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children
    children = data
    data = undefined
  }
  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE
  }
  return _createElement(context, tag, data, children, normalizationType)
}
```

每个VNode有children，children每个元素也是一个VNode，这样就形成了一个VNode Tree，它很好的描述了我们的DOM Tree。

通过vm._update把VNode渲染成真实DOM。它被调用的时机有2个，一个是首次渲染，一个是数据更新的时候。_update的核心方法是`vm.__patch__`。

从初始化Vue到最终渲染的整个过程，参考下图：

![示例](./imgs/img6.png)


> 这个笔记是参考此[文档](https://ustbhuangyi.github.io/vue-analysis/v2/prepare/)整理记录搬运，原文档非常赞。感谢作者🙏非常赞。
