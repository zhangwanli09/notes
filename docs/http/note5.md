# 跨域资源共享（CORS）

[CORS](https://developer.mozilla.org/zh-CN/docs/Glossary/CORS) 是一种基于 HTTP 头的机制，该机制通过允许服务器标示除了它自己以外的其它 origin（域，协议和端口），这样浏览器可以访问加载这些资源。解决浏览器 [同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy) 引起的`跨域`限制。

使用`Origin`和`Access-Control-Allow-Origin`就能完成最简单的访问控制。

部分响应头：

```
Access-Control-Allow-Origin: <origin> | *
```

[Access-Control-Allow-Origin](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) 指定了该响应的资源是否被允许与给定的`origin`共享。其中，origin 参数的值指定了允许访问该资源的外域 URI。对于`不需要`携带身份`凭证`的请求，服务器可以指定该字段的值为`通配符`，表示允许来自所有域的请求。对于附带身份凭证（Cookie）的请求，服务器不得设置该字段的值为`*`。

如果服务端指定了`具体`的域名而非`*`，那么响应首部中的`Vary`字段的值必须包含`Origin`。这将告诉客户端：服务器对不同的源站返回不同的内容。

```
Access-Control-Allow-Origin: https://developer.mozilla.org
Vary: Origin
```

请求头：

```
Origin: <origin>
```

[origin](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Origin) 参数的值为源站 URI。它不包含任何路径信息，只是服务器名称。

注意，在所有访问控制请求（Access control request）中，Origin 首部字段总是被发送。

> [MDN 跨域资源共享（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)
