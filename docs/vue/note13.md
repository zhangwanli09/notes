# 组件通信

1. props / $emit
2. $emit / $on
3. $parent。子实例访问父实例
4. $children。父实例访问子实例
5. $refs。访问组件实例
6. $attrs。包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。
7. $listeners。包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。
8. provide / inject。这对选项需要一起使用，允许祖先组件向其所有子孙后代注入依赖，不论组件层次有多深。祖先组件通过provider提供变量给子孙组件，子孙组件通过inject注入变量给祖先组件。
9. Vuex

> [参考](https://segmentfault.com/a/1190000022083517)
> 
> [参考](https://cn.vuejs.org/v2/api/#provide-inject)
