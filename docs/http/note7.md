# HTTP/2 的多路复用

在 HTTP/1.x 中，每次请求都会建立一次 TCP 连接（3 次握手和 4 次挥手），这个过程很耗时，即便开启 [Keep-Alive](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Keep-Alive)，解决了多次连接的问题，但依然有两个效率上的问题：

1. 串行的文件传输。
2. 连接数过多导致的性能问题。

HTTP/2 的`多路复用`就是为了解决上述的两个性能问题。

在 HTTP/2 中，有两个非常重要的概念（`二进制分帧`）：`帧`（frame）和`流`（stream）：

1. `帧`：代表最小的数据单位，多个帧之间可以乱序发送，根据帧首部的流标识可以重新组装。
2. `流`：多个帧组成的数据流。存在于连接中的一个虚拟通道。流可以承载双向消息，每个流都有一个唯一的整数ID。

`多路复用`，就是在`一个 TCP`连接中可以存在`多条流`。换句话说，就是可以发送`多个`请求，对端可以通过`帧`中的`标识`知道属于哪个请求。通过这个技术，可以避免 HTTP 旧版本中的`队头阻塞`问题，极大的提高传输性能。