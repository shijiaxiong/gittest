## Q:HTTP与HTTPS的区别？

- HTTP和HTTPS都属于应用层协议。TCP属于传输层，SSL属于会话层。
- HTTP 协议运行在TCP上，明文传输，客户端和服务端无法验证对方身份。HTTPS是身披SSL外壳的HTTP，运行在SSL上，SSL运行在TCP上，是添加了加密和验证机制的HTTP。
- **端口：** HTTP使用80端口，HTTPS使用443。
- **资源开销：** 和HTTP相比，HTTPS会因为加解密消耗更多的CPU和资源。

![](./static/164e529309f0fa33)



## Q:HTTP和TCP的区别？

- TCP是传输层协议，保证数据的可靠性传输。
- HTTP是应用层协议，建立在TCP连接之上，主要规定客户端和服务端的数据传输格式。



## Q:http 1.0 1.1 2.0 区别?

#### HTTP 1.0

- 默认87不支持长连接，长连接需要设置`Connection: keep-alive`指定。
- **缓存处理：** 使用`header`里的`If-Modified-Since,Expires`来做为缓存判断的标准。

#### HTTP 1.1

- 默认支持长连接，多个http请求可以复用一个TCP连接，但是同一时间只能对应一个http请求(http请求在一个Tcp中是串行的)。
- **Host头处理** 在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。
- 引入了更多的缓存控制策略例如`Entity tag`，`If-Unmodified-Since`, `If-Match`, `If-None-Match`等更多可供选择的缓存头来控制缓存策略。

#### HTTP 2.0 （与HTTP1.X的区别）

- 默认支持长连接。
- 多路复用，一个Tcp中多个http请求是并行的 (雪碧图、多域名散列等优化手段http/2中将变得多余)
- 使用`HAPCK`算法对header数据进行压缩，使数据体积变小，传输更快
- 支持服务器端推送。
- 协议解析使用二级制格式，方便健壮。



## Q:HTTPS握手的五个阶段(HTTPS建立连接的过程)

- 客户端向服务器发送支持的SSL/TSL的协议版本号，以及客户端支持的加密方法，和一个客户端生成的随机数。
- 服务器确认协议版本号和加密方法，向客户端发送一个由服务器生成的随机数，以及数字证书。
- 客户端验证证书是否有效，有效则从证书中取出公钥，生成一个随机数，然后用公钥加密这个随机数，发给服务器。
- 服务器用私钥解密，获取客户端发来的随机数。
- 客户端和服务器根据预定好的加密方法，使用前面生成的三个随机数，生成对话密钥，用来加密接下来的对话过程。

#### Reference

[https 原理](https://app.yinxiang.com/shard/s43/nl/13675070/cff50944-5c13-4ef9-8ca8-95de0d79d10a)



## Q:用netstat看tcp连接的时候有关注过time_wait和close_wait吗？

- `TIME_WAIT`产生的点：主动关闭连接的一方收到FIN包，会回复ACK，进入`TIME_WAIT`状态。等待2MSL时间结束。
- `CLOSE_WAIT` 产生的点：被动关闭的一方收到FIN包后，协议层回复ACK，然后进入`CLOSE_WAIT`状态，等待应用程序调用close()



#### CLOSE_WAIT过多的原因

- 程序有问题，没有合适的关闭socket。
- 服务器CPU处理不过来
- 应用程序一直睡眠或者阻塞在其他地方(锁等待、I/O操作等)，使得程序得不到调度，没法执行close操作。



#### TIME_WAIT过多的原因

- 在高并发短连接的TCP服务上，服务器处理完请求后主动关闭连接。大量的SOCKET会处于TIME-WAIT阶段。
- 如果客户端的并发量持续很高，此时部分客户端就会显示连接不上。

解决方案

- 增加服务器。
- 尽可能使用长连接。
- 修改内核参数，打开系统的TIMEWAIT重用和快速回收。



#### Reference

[TCP连接的TIME_WAIT和CLOSE_WAIT ](https://app.yinxiang.com/shard/s43/nl/13675070/092eb018-7063-4a88-9bdb-5a9fa96c6709)





## Q:TCP连接意外中断？

- 检测方式：TCP实现的keepalive，开启需要消耗额外的带宽和流量，默认是关闭状态。另一种是应用层实现心跳包。

[TCP连接意外中断怎么检测](https://blog.csdn.net/shayne000/article/details/95894135)









