# 安全相关

## 跨站脚本攻击（XSS）

[XSS](https://developer.mozilla.org/zh-CN/docs/Glossary/Cross-site_scripting) 是一种安全漏洞攻击，攻击者可以利用这种漏洞在网站上`注入恶意代码`。当被攻击者登陆网站时就会自动运行这些恶意代码，从而攻击者可以突破网站的访问权限，冒充受害者。[维基百科 XSS](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%B6%B2%E7%AB%99%E6%8C%87%E4%BB%A4%E7%A2%BC)

容易发生 XSS 攻击的 2 种情况：

1. 数据从一个`不可靠的链接`进入 Web 应用。
2. 没有过滤掉恶意代码的`动态内容`被发送给 Web 用户。

恶意内容一般包括 JS、HTML 或是其他浏览器可执行的代码。他们通常都会：将`cookies`或其他隐私信息发送给攻击者，将受害者重定向到由攻击者控制的网页，或是经由恶意网站在受害者的机器上进行其他恶意操作。

XSS 攻击可分为 3 类：存储型、反射型、DOM 型。

防御措施：

1. 过滤特殊字符。将用户所提供的内容进行过滤。
2. 设置`HttpOnly`类型的 Cookie。阻止 JS 对其访问。
3. 设置响应头 [Content-Security-Policy](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy)。（参考 CSP）


## 跨站请求伪造（CSRF）

[CSRF](https://developer.mozilla.org/zh-CN/docs/Glossary/CSRF) 冒充受信任用户，向服务器发送请求。

CSRF 的本质在于攻击者欺骗用户去访问自己设置的地址。这利用了 web 中用户身份验证的一个漏洞：简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的。[维基百科 CSRF](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0)

跟 跨站脚本攻击（XSS）相比，`XSS`利用的是用户对指定网站的信任，`CSRF`利用的是网站对用户网页浏览器的信任。

例如，这些非预期请求可能是通过在跳转链接后的 URL 中加入恶意参数来完成：

```html
<img src="https://www.example.com/index.php?action=delete&id=123">
```

防御措施：

1. 添加 token 校验。
2. 检查 Referer 字段（同源检测）。
3. Set-Cookie 时设置 SameSite=Lax，在`跨站请求`时不会被发送。

## 内容安全策略（CSP）

[CSP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP) 是一个额外的安全层，用于检测并削弱某些特定类型的攻击（XSS，数据注入攻击等）。它的本质是建立一个白名单，告诉浏览器哪些外部资源可以加载和执行。我们只需要配置规则，如何拦截由浏览器自己来实现。

两种方式来开启 CSP：

1. 配置 [Content-Security-Policy](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy) HTTP 头。
2. 配置 `<meta>` 标签。

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src https://*; child-src 'none';">
```

CSP 的主要目标是减少和报告`XSS`攻击。

## 中间人攻击（MITM）

中间人攻击（Man-in-the-middle attack，MITM）会在消息发出方和接收方之间`拦截双方通讯`。[维基百科 中间人攻击](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB)

防御措施：

1. 不要忽视浏览器弹出的证书警告！你可能访问的是钓鱼网站或者假冒的服务器。
2. 公共网络环境下（例如公共WiFi），没有`HTTPS`加密的敏感网站不要随便登录，一般不可信。
3. 在任何网站上登录自己的账号前确保网址是走的`HTTPS`加密协议。
