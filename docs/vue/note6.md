# Vuex

Vuex 是一个专为 Vue.js 应用程序开发的`状态管理模式`。它采用`集中式存储`管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

## 状态管理模式

1. `state`，驱动应用的数据源；
2. `view`，以声明方式将 state 映射到视图；
3. `actions`，响应在 view 上的用户输入导致的状态变化。

单向数据流：

![单向数据流](https://ustbhuangyi.github.io/vue-analysis/assets/vuex.png ':size=500')

当应用遇到多个组件共享状态时，单向数据流的简洁性很容易被破坏：

1. 多个视图依赖于同一状态。
2. 来自不同视图的行为需要变更同一状态。

解决思路：抽取组件的共享状态，以一个全局单例模式管理。

## Vuex 基本思想

Vuex 应用的核心是 store。它包含应用中大部分的状态（state）。

1. Vuex 的状态存储是`响应式`的。当 store 中的状态发生变化时，相应的组件也会高效更新。
2. 不能直接改变 store 中的状态。更改 store 中的状态的唯一方法是提交`mutation`。

通过定义和隔离状态管理中的各种概念并通过强制规则维持视图和状态间的独立性，我们的代码将会变得更结构化且易维护。

![Vuex](https://vuex.vuejs.org/vuex.png ':size=600')

## 核心概念

1. [State](https://vuex.vuejs.org/zh/guide/state.html)
2. [Getter](https://vuex.vuejs.org/zh/guide/getters.html)
3. [Mutation](https://vuex.vuejs.org/zh/guide/mutations.html)
4. [Action](https://vuex.vuejs.org/zh/guide/actions.html)
5. [Module](https://vuex.vuejs.org/zh/guide/modules.html)

> [Vuex](https://vuex.vuejs.org/zh/)
