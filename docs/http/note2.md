# HTTP Headers

HTTP 消息头允许客户端和服务器通过`request`和`response`传递附加信息。请求头名称`不区分`大小写。通过 HTTP 消息头，可以使服务器或客户端了解对方所使用的协议版本、内容类型、编码方式等。参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)。

消息头分为：

1. General headers：`通用头`，同时适用于请求和响应消息，但与最终消息主体中传输的数据无关的消息头。
2. Request headers：`请求头`，包含更多有关要获取的资源或客户端本身信息的消息头。
3. Response headers：`响应头`，包含有关响应的补充信息，如其位置或服务器本身（名称和版本等）的消息头。
4. Entity headers：`实体头`，包含有关实体主体的更多信息，比如主体长(Content-Length)度或其MIME类型。


常见的字段：

Request headers（请求头）：

1. Accept：可接受的响应内容类型（Content-Types）。示例：Accept: text/plain
2. Accept-Charset：可接受的字符集。示例：Accept-Charset: utf-8
3. `Accept-Encoding`：可接受的响应内容的编码方式。示例：Accept-Encoding: gzip, deflate
4. Accept-Language：可接受的响应内容语言列表。示例：Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
5. Authorization：用于表示HTTP协议中需要认证资源的认证信息。示例：Authorization: Basic OSdjJGRpbjpvcGVu..
6. `Cache-Control`：用来指定当前的请求/回复中的，是否使用缓存机制。示例：Cache-Control: no-cache
7. Connection：客户端（浏览器）想要优先使用的连接类型。示例：Connection: keep-alive
8. `Cookie`：由之前服务器通过 Set-Cookie 设置的一个HTTP协议Cookie。示例：Cookie: $Version=1; Skin=new;
9. Content-Length：以8进制表示的请求体的长度。示例：Content-Length: 348
10. Content-Type：请求体的MIME类型 （用于POST和PUT请求中）。示例：Content-Type: application/x-www-form-urlencoded
11. Host：服务器的域名以及服务器所监听的端口号。如果所请求的端口是对应的服务的标准端口（80），则端口号可以省略。示例：Host: news.baidu.com
12. `If-Modified-Since`：允许在对应的资源未被修改的情况下返回304未修改。
13. `Origin`：发起一个针对跨域资源共享的请求（该请求要求服务器在响应中加入一个Access-Control-Allow-Origin的消息头，表示访问控制所允许的来源）
14. Range：表示请求某个实体的一部分，字节偏移以0开始。示例：Range: bytes=500-999
15. Referer：表示浏览器所访问的前一个页面，可以认为是之前访问页面的链接将浏览器带到了当前页面。
16. `User-Agent`：浏览器的身份标识字符串。示例：User-Agent: Mozilla/5.0...

Response headers（响应头）：

1. `Access-Control-Allow-Origin`：指定哪些网站可以跨域源资源共享。示例：Access-Control-Allow-Origin: *
2. Accept-Patch：指定服务器所支持的文档补丁格式。示例：Accept-Patch: text/example;charset=utf-8
3. Accept-Ranges：服务器所支持的内容范围。示例：Accept-Ranges: bytes
4. Age：响应对象在代理缓存中存在的时间，以秒为单位。示例：Age: 12
5. Allow：对于特定资源的有效动作。示例：Allow: GET, HEAD
6. `Cache-Control`：通知从服务器到客户端内的所有缓存机制，表示它们是否可以缓存这个对象及缓存有效时间。其单位为秒。Cache-Control: max-age=3600
7. `Connection`：针对该连接所预期的选项。示例：Connection: close
8. Content-Disposition：对已知MIME类型资源的描述，浏览器可以根据这个响应头决定是对返回资源的动作，如：将其下载或是打开。
9. `Content-Encoding`：响应资源所使用的编码类型。示例：Content-Encoding: gzip
10. Content-Language：响就内容所使用的语言。示例：Content-Language: zh-cn
11. Content-Length：响应消息体的长度，用8进制字节表示。示例：Content-Length: 348
12. Content-Range：如果是响应部分消息，表示属于完整消息的哪个部分。Content-Range: bytes 21010-47021/47022
13. Content-Type：当前内容的MIME类型。示例：Content-Type: text/html; charset=utf-8
14. Date：此条消息被发送时的日期和时间。
15. `Location`：用于在进行重定向，或在创建了某个新资源时使用。
16. Refresh：用于重定向，或者当一个新的资源被创建时。默认会在5秒后刷新重定向。
17. `Server`：服务器的名称。示例：Server: nginx/1.6.3
18. `Set-Cookie`：设置HTTP cookie。示例：Set-Cookie: UserID=itbilu; Max-Age=3600; Version=1
19. `Status`：通用网关接口的响应头字段，用来说明当前HTTP连接的响应状态。示例：Status: 200 OK
20. WWW-Authenticate：表示在请求获取这个实体时应当使用的认证模式。示例：WWW-Authenticate: Basic

> 参考：https://itbilu.com/other/relate/EJ3fKUwUx.html