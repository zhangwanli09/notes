# Context

[Context](https://zh-hans.reactjs.org/docs/context.html) 提供了一种在组件之间`共享数据`（主题，用户信息，地区偏好等数据）的方式，`避免`组件间`逐层传递 props`的繁琐过程。

应用场景：很多不同层级的组件需要访问相同的数据。请谨慎使用，因为这会使得组件的`复用性变差`。如果你只是想避免层层传递一些属性，[组件组合（component composition）](https://zh-hans.reactjs.org/docs/composition-vs-inheritance.html) 有时候是一个比 context 更好的解决方案。

示例：

```javascript
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext = React.createContext('light');
class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    // 无论多深，任何组件都能读取这个值。
    // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // 指定 contextType 读取当前的 theme context。
  // React 会往上找到最近的 theme Provider，然后使用它的值。
  // 在这个例子中，当前的 theme 值为 “dark”。
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```

[API](https://zh-hans.reactjs.org/docs/context.html#api)：

1. React.createContext
2. Context.Provider
3. Class.contextType
4. Context.Consumer
5. Context.displayName
