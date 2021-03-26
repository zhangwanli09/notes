# TCP 连接

在真正的读写操作之前，server 与 client 之间必须建立一个连接，当读写操作完成后，双方不再需要这个连接时，它们可以释放这个连接，连接的`建立`是需要`3 次握手`，而`释放`则需要`4 次握手`，所以每次连接都需要消耗资源和时间。

TCP 属于`传输层`，提供可靠的字节流服务。

1. 字节流服务。将大块的数据`分割`成以报文段为单位的`数据包`进行管理。
2. 传输服务。把数据`准确`可靠的传输给对方。

TCP 协议为了更容易传送大数据才把数据分割，且能够确认数据最终是否送达到对方（三次握手）。

握手过程使用 TCP 的标志：`SYN`（synchronize）和`ACK`（acknowledgement）：

1. 客户端发送一个带`SYN`标志的数据包给服务器。
2. 服务器收到后，回传一个带有`SYN/ACK`标志的数据包给客户端。
3. 客户端再回传一个带`ACK`标志的数据包给服务器。握手结束。

![三次握手](https://www.ituring.com.cn/figures/2014/PIC%20HTTP/05.d01z.009.png ':size=600')