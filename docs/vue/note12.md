# diff

### 当数据变化时，Vue是怎么更新节点的？

渲染真实dom开销很大，会引起整个dom树回流重绘。diff算法可以解决只更新需要修改的部分而不需要更新整个dom。

先根据真实DOM生成`virtual DOM`，当`virtual DOM`上某个节点发生变化后会生成新的`vnode`，然后`vnode`和`oldVnode`作对比，发现有不一样的地方就直接修改在真实dom上，然后使`oldVnode`的值为`vnode`。

`diff`的过程就是调用名为`patch`的函数，比较`新旧节点`，一边比较一边给真实的DOM打补丁。


### virtual DOM和真实DOM的区别？

virtual DOM是将真实的DOM的`数据抽取`出来，以`对象`形式模拟树形结构。

`VNode`和`oldVNode`都是对象。

### diff的比较方式

在比较新旧节点时，比较只会在同层级发生，不会跨层级比较。

> [参考](https://juejin.cn/post/6844903607913938951)
