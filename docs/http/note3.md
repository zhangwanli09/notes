# HTTP Cookie

HTTP Cookie（也叫 Web Cookie 或浏览器 Cookie）是服务器发送到用户浏览器并`保存在本地`的一小块数据，它会在浏览器下次向同一服务器再发起请求时`被携带`并发送到服务器上。

通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的`登录状态`。Cookie 使基于无状态的HTTP协议记录稳定的状态信息成为了可能。

Cookie 主要用于：
1. 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
2. 个性化设置（如用户自定义设置、主题等）
3. 浏览器行为跟踪（如跟踪分析用户行为等）

Cookie 曾被用于客户端数据存储，但服务器指定 Cookie 后，每次请求都会携带 Cookie 数据，会带来额外的`性能开销`（尤其是移动端）。随着现代浏览器开始支持多种存储方式（`Web storage`，IndexedDB），Cookie 渐渐被淘汰。

### 创建 Cookie

当服务器收到 HTTP 请求时，服务器可以在响应头里面添加一个`Set-Cookie`选项。浏览器收到响应后通常会保存下 Cookie，之后对该服务器每一次请求中都通过`Cookie`请求头部将 Cookie 信息发送给服务器。另外，Cookie 的过期时间、域、路径、有效期、适用站点都可以根据需要来指定。

```
Set-Cookie: <cookie名>=<cookie值>
```

服务器通过该头部告知客户端保存 Cookie 信息。

```
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry
```

现在，对该服务器发起的每一次新请求，浏览器都会将之前保存的Cookie信息通过 Cookie 请求头部再发送给服务器。

```
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```

### 定义 Cookie 的生命周期

Cookie 的生命周期可以通过`两种`方式定义：

1. `会话期 Cookie` 是最简单的 Cookie：浏览器`关闭`之后它会被`自动删除`，也就是说它仅在会话期内有效。会话期 Cookie 不需要指定过期时间（Expires）或者有效期（Max-Age）。
2. `持久性 Cookie` 的生命周期取决于过期时间（Expires）或有效期（Max-Age）指定的一段时间。

```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;
```

当 Cookie 的过期时间被设定时，设定的日期和时间只与客户端相关，而不是服务端。

如果站点对用户进行身份验证，则每当用户进行身份验证时，它都应重新生成并重新发送会话 Cookie，甚至是已经存在的会话 Cookie。此技术有助于防止`会话固定攻击`（session fixation attacks），在该攻击中第三方可以重用用户的会话。

### 限制访问 Cookie

有`两种`方法可以确保 Cookie 被`安全发送`，并且不会被意外的参与者或脚本访问：

1. `Secure`属性。预防`中间人（man-in-the-middle）`攻击。
2. `HttpOnly`属性。预防`跨站点脚本（XSS）`攻击。

标记为`Secure`的 Cookie 只应通过被`HTTPS`协议加密过的请求发送给服务端，因此可以预防`中间人（man-in-the-middle）`攻击者的攻击。但即便设置了 Secure 标记，敏感信息也不应该通过 Cookie 传输，因为 Cookie 有其固有的不安全性，Secure 标记也无法提供确实的安全保障, 例如，可以访问客户端硬盘的人可以读取它。从 Chrome 52 和 Firefox 52 开始，不安全的站点（http:）无法使用Cookie的 Secure 标记。

JavaScript `Document.cookie` API `无法访问`带有`HttpOnly`属性的 cookie，此类 Cookie 仅作用于服务器。例如，持久化服务器端会话的 Cookie 不需要对 JavaScript 可用，而应具有 HttpOnly 属性。此预防措施有助于缓解`跨站点脚本（XSS）`攻击。

```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```

### Cookie 的作用域

`Domain`和`Path`标识定义了 Cookie 的作用域：即允许 Cookie 应该发送给哪些URL。

1. `Domain`指定了哪些主机可以接受 Cookie。如果不指定，默认为 origin，不包含子域名。如果指定了Domain，则一般包含子域名。当子域需要共享有关用户的信息时，例如，如果设置 Domain=mozilla.org，则 Cookie 也包含在子域名中（如developer.mozilla.org）。

2. `Path`指定了主机下的哪些路径可以接受 Cookie（该 URL 路径必须存在于请求 URL 中）。以字符 %x2F ("/") 作为路径分隔符，子路径也会被匹配。例如，设置 Path=/docs，则 /docs、/docs/Web/、/docs/Web/HTTP 都会匹配。

### SameSite 属性

`SameSite`：SameSite Cookie 允许服务器要求某个 cookie 在`跨站请求`时不会被发送，（其中 Site 由可注册域定义），从而可以阻止`跨站请求伪造（CSRF）`攻击。

```
Set-Cookie: key=value; SameSite=Strict
```

SameSite 有三种值：

1. `None`。浏览器会在同站请求、跨站请求下继续发送 cookies，不区分大小写。
2. `Strict`。浏览器将只在访问相同站点时发送 cookie。（在原有 Cookies 的限制条件上的加强，如上文 Cookie 的作用域所述）。
3. `Lax`。与 Strict 类似，但用户从外部站点导航至URL时（例如通过链接）除外。在新版本浏览器中，为`默认`选项，Same-site cookies 将会为一些跨站子请求保留，如图片加载或者 frames 的调用，但只有当用户从外部站点导航到URL时才会发送。如 link 链接。

### Cookie 前缀
