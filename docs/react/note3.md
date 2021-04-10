# 高阶组件（HOC）

[高阶组件（HOC）](https://zh-hans.reactjs.org/docs/higher-order-components.html) 是 React 中用于`复用组件逻辑`的一种高级技巧。`高阶组件`是参数为组件，返回值为新组件的函数。

```javascript
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

组件是将 props 转换为 UI，而高阶组件是将组件转换为`另一个组件`。（例如 Redux 的 connect）

允许我们在一个地方定义这个逻辑，并在许多组件之间共享它。这正是高阶组件擅长的地方。
