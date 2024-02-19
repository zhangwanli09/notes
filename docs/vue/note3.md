# 生命周期

每个 Vue 实例在被创建时都要经过一系列的初始化过程——例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等。同时在这个过程中也会运行一些叫做`生命周期钩子`的函数，这给了用户在不同阶段添加自己的代码的机会。

`不要`在选项 property 或回调上使用`箭头函数`，因为箭头函数并没有 this，this 会作为变量一直向上级词法作用域查找，直至找到为止。

生命周期图：

![生命周期图](https://cn.vuejs.org/images/lifecycle.png ':size=600')

## 父组件和子组件生命周期钩子执行顺序

1. 加载渲染过程：父 beforeCreate -> 父 created -> 父 beforeMount -> 子 beforeCreate -> 子 created -> 子 beforeMount -> 子 mounted -> 父 mounted。
2. 子组件更新过程：父 beforeUpdate -> 子 beforeUpdate -> 子 updated -> 父 updated。
3. 父组件更新过程：父 beforeUpdate -> 父 updated。
4. 销毁过程：父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed。

> [实例生命周期钩子](https://cn.vuejs.org/v2/guide/instance.html#%E5%AE%9E%E4%BE%8B%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E9%92%A9%E5%AD%90)