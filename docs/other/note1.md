# 浏览器工作原理

## 导航

导航是加载页面的第一步。用户输入URL，点击链接或是提交表单等。web性能优化的目标之一就是缩短导航花费的时间。

1. DNS查找。
    1. 通过DNS查找返回IP地址，第一次请求后，这个IP可能会被缓存，这样就可以在缓存里检索IP而不用再次查找，加速后续请求（DNS缓存机制）。
2. TCP握手。
    1. 获取到IP地址后，浏览器会通过TCP三次握手与服务器建立连接。这个机制是用来让两端尝试进行通信。
    2. TCP三次握手技术（SYN, SYN-ACK, ACK）。通过TCP首先发送三个消息进行协商（在两台电脑之前开始一个TCP会话）。意味着每台服务器之间还要来回发送三条消息，而请求尚未发出。
3. TLS协商。
    1. 在进行真实数据传出之前建立安全连接。


## 响应

建立web服务器连接之后，浏览器会发送一个初始HTTP GET请求（通常是HTML文件）。服务器收到请求后使用相关的响应头和HTML内容进行回复。

1. TCP慢开始/14kb。
    1. 第一个响应包大小是14kb，慢开始逐渐增加发送数据的数量。
    2. 在收到初始包（14kb）后，服务器会将下一个包大小加倍到大约28kb。后续包依次是前一个包的2倍，直到预定阀值或者遇到拥塞。
2. 拥塞控制。
    1. 当服务器用TCP包发送数据时，客户端通过返回确认帧来确认传输。
    2. 由于网速原因，如果服务器太快发送过多的包，就可能会丢包。意味着将不会有确认帧返回。服务器吧它们当作确认帧丢失。拥塞控制算法使用这个发送包和确认帧流来确认发送速率。


## 解析

解析是浏览器将收到的数据转换为DOM和CSSOM的步骤，通过渲染器把DOM和CSSOM绘制成页面。

从初始包（14kb）就开始解析并尝试渲染。这也说明为什么在前14kb中包含浏览器开始渲染页面所需的所有内容，或者至少包含模版对于web性能优化非常重要。

1. 构建DOM树。
    1. DOM树反应节点间的关系和层级结构。
    2. DOM节点数越多，构建树的时间越长。文档格式良好，解析的更快。
    3. 当解析器发现非阻塞资源（图片，css），浏览器会请求这些资源并继续解析。但是遇到script标签（特别是没有async或defer属性）会阻塞渲染并停止解析（虽然浏览器预加载扫描器会加速这过程，但还是有瓶颈）。
2. 预加载扫描器。
    1. 构建DOM树时，预加载扫描器将解析可用的内容并请求优先级高的资源（css，js，web字体）。它在后台检索资源，解析器达到请求的资源时，它们可能已经在运行或者已下载。预加载扫描器提供优化并减少阻塞。
3. 构建CSSOM树。
    1. DOM和CSSOM是两个树，独立的数据结构。
    2. 构建CSSOM树速度非常快（性能优化基本可以忽略此项）。通常小于1次DNS查找时间。
4. 其他过程。
    1. js编译。


## 渲染

将DOM树和CSSOM树组合成一个Render树（渲染树），用来计算每个可见元素的布局，然后绘制到屏幕。

某些情况下，可以将内容通过GPU而不是CPU上绘制部分内容来提高性能，释放主线程。

1. Style。
    1. 从DOM树根节点开始构建，便利每个可见节点。
    2. 每个可见节点应用CSSOM树规则。
    3. Render树保存所有具有内容和计算样式的可见节点。
2. 布局。
    1. 构建渲染树后，开始布局（位置，大小，尺寸等）。
    2. 浏览器从渲染树的根节点开始遍历。
    3. 第一次确定节点的大小和位置成为布局。随着节点大小和位置的变动会重新计算（回流）。
3. 绘制。
    1. 浏览器将节点绘制到屏幕。
    2. 绘制可以将布局元素分解成多层。将内容提升到GPU层（而不是CPU主线程）可以提高绘制和重绘性能。
    3. 特定的属性和元素可以实例化一个层（video，canvas标签，opacity，transform，transition等）。
    4. 层虽然可以提高性能，但它以内存管理为代价，因此不应作为web性能优化策略过度使用。
4. 合成。
    1. 当文档的各个部分以不同层绘制，相互重叠时，必须进行合成，确保以正确的顺序绘制到屏幕。
    2. 页面加载中发生回流也会触发重绘和重新合成。


> 参考：
>
> [MDN](https://developer.mozilla.org/zh-CN/docs/Web/Performance/How_browsers_work)
>
> [浏览器的工作原理](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/)
