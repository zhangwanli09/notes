# 浏览器多个页签之间通讯

本质上都是通过中介者模式实现。因为标签之前没办法直接通信，因此可以找个中介者，让标签页和中介者通信，然后通过中介进行消息转发。

1. 使用WebSocket，要通信的标签页连接同一个服务器，发送消息到服务器后，服务器推送消息给所有连接的客户端。
2. 使用localStorage，在另一个浏览上下文里被添加，修改或删除时，会触发storage事件，通过监听storage事件进行信息通信。
3. 如果能获得对应页签的引用，通过postMessage()方法也可以实现。


## postMessage

window.postMessage()可以安全实现跨源通信。

1. 获得对另一个窗口的引用（比如targetWindow = window.opener）。
2. 然后在窗口上调用targetWindow.postMessage()方法分发一个MessageEvent消息。
3. 接收消息的窗口可以根据需要自由处理此事件。传递给window.postMessage()的参数将通过消息事件对象暴漏给接收消息的窗口。

参考：https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage
