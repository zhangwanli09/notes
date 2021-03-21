# 为什么v-for中的key不推荐使用随机数或者index

因为在插入或删除数据时，会导致后面的数据的key绑定的index变化，导致重新渲染，效率会降低。

使用唯一的key，便于`diff`更`高效`的进行节点`查询对比`。能更快的找到需要更新的dom。还有就是`防止复用`dom。

有key时，通过`createKeyToOldIdx`方法生成key与索引`映射`关系，直接通过新子节点的key查询是否存在于旧子节点序列中。

无key时，必须遍历旧子节点序列，依次与新子节点对比判断是否为新增节点。

源码：

```javascript
// 生成 key 与索引映射关系
function createKeyToOldIdx (children, beginIdx, endIdx) {
  let i, key
  const map = {}
  for (i = beginIdx; i <= endIdx; ++i) {
    key = children[i].key
    if (isDef(key)) map[key] = i
  }
  return map
}

// 比较 key
function sameVnode (a, b) {
  return (
    a.key === b.key && (
      (
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        isDef(a.data) === isDef(b.data) &&
        sameInputType(a, b)
      ) || (
        isTrue(a.isAsyncPlaceholder) &&
        a.asyncFactory === b.asyncFactory &&
        isUndef(b.asyncFactory.error)
      )
    )
  )
}
```

参考源码 [patch.js](https://github.com/vuejs/vue/blob/v2.6.11/src/core/vdom/patch.js) 中`updateChildren`方法。
