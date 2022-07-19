# 动态组件 & 异步组件

### 动态组件

通过Vue的`<component>`元素加`is`属性来实现在不同组件之间进行`动态切换`。

```html
<!-- 组件会在 `currentTabComponent` 改变时改变 -->
<!-- currentTabComponent 是已注册的组件名 -->
<component v-bind:is="currentTabComponent"></component>
```

当在这些组件之间切换的时候，你有时会想`保持组件状态`，以避免反复重新渲染导致的性能问题。可使用`<keep-alive>`包裹。

```html
<!-- 失活的组件将会被缓存！-->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

### 异步组件

只在需要的时候才加载。
