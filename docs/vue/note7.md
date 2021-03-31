# history 与 hash 路由的区别

前端路由的核心是：改变视图的同时不会向后端发出请求。为了达到这一目的，浏览器提供了`hash`和`history`两种模式。

vue-router 默认 hash 模式。使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。

也可以用路由的 history 模式，这种模式充分利用 [history.pushState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState) API 来完成 URL 跳转而无须重新加载页面。

1. `hash`。hash 虽然出现在 URL 中，但不会被包含在 http 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面。
2. `history`。[history](https://developer.mozilla.org/zh-CN/docs/Web/API/History) 利用了 html5 history interface 中新增的`pushState()`和`replaceState()`方法。这两个方法应用于浏览器记录栈，在当前已有的 back、forward、go 基础之上，它们提供了`对历史记录修改`的功能。只是当它们执行修改时，虽然改变了当前的 URL ，但浏览器不会立即向后端发送请求。

因此可以说， hash 模式和 history 模式都属于浏览器自身的属性，`vue-router`只是利用了这两个特性（通过调用浏览器提供的接口）来实现路由。

实现原理：

1. `hash`：原理是 [onhashchange](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowEventHandlers/onhashchange) 事件，可以在`window`对象上监听这个事件。
2. `history`：hashchange 只能改变 # 后面的代码片段，[history](https://developer.mozilla.org/zh-CN/docs/Web/API/History) api（pushState、replaceState、go、back、forward）则给了前端完全的自由，通过在`window`对象上监听`popState()`事件。


使用`history.pushState()`的优势：

1. pushState 设置的 url 可以是同源下的任意 url。而 hash 只能修改 # 后面的部分，因此只能设置当前 url 同文档的 url。
2. pushState 设置的新的 url 可以与当前 url 一样，这样也会把记录添加到栈中。hash 设置的新值不能与原来的一样，一样的值不会触发动作将记录添加到栈中。
3. pushState 通过 stateObject 参数可以将任何数据类型添加到记录中。hash 只能添加短字符串。
4. pushState 可以设置额外的 title 属性供后续使用。

它也有些需要注意的点：

1. history 在刷新页面时，如果服务器中没有相应的响应或资源，就会出现 404。因此，如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。
2. hash 模式下，仅 # 之前的内容包含在 http 请求中，对后端来说，即使没有对路由做到全面覆盖，也不会报 404。

[参考](https://juejin.cn/post/6844903625169502216)，[Vue Router](https://router.vuejs.org/zh/)
