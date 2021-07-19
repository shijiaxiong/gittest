## Q:HTTP 报文

- HTTP报文是由多行数据结构构成的字符串文本。行与行之间使用空行（回车符\r 和换行符\n）来划分。

- HTTP报文大致分为报文首部和报文主体两块。使用空行划分。
- 报文首部是服务端或客户端处理的请或响应的内容及属性。
- 报文主体是应发送的数据。

## Q:响应状态码？

- 1xx（信息）：收到请求，接收的请求正在处理。
- 2xx（成功）：请求已成功处理。
- 3xx（重定向）：需要采取进一步措施才能完成请求。
- 4xx（客户端错误）：请求包含错误的语法或参数不满足，服务器无法处理请求。
- 5xx（服务器错误）：服务器处理请求出错。



#### 常见状态码

- 200 OK 请求正常处理。

- 204 请求处理成功 但是没有任何资源返回给客户端(一般用于只需客户端向服务端发送消息)。

- 206 对资源的某一部分请求 响应报文中包含由 Content-Range 指定范围的实体内容。



- 301（永久重定向） 如果把资源对应的URI保存为书签，则此时书签会根据Localtion首部字段提示的URI重新保存。
- 302 （临时重定向） 临时地从旧地址A跳转到地址B。



- 400（错误请求） 请求报文内有语法错误。

- 401（未授权）请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应。

- 403（禁止） 请求资源的访问被服务器拒绝，一般是未获得文件系统的访问权限，访问权限出现问题。

- 404 （未找到）服务器上找不到请求资源 或路径错误。

- 405 （方法禁用）请求方法被服务端识别，但是服务端禁止使用该方法。可以用OPTIONS来查看服务器允许哪些访问方法。



- 500（服务器内部错误） 服务器端在执行请求时出错，一般是因为web应用出现bug。
- 502（错误网关） 代理服务器或网关从上游服务器中收到无效响应。
- 503（服务不可用） 服务器暂时处于超负载或停机维护，目前无法处理请求。
- 504 （网关超时）服务器作为网关或代理，但是没有及时从上游服务器收到请求。



## Q：GET 和 POST的区别？

- 功能上：GET获取资源，POST修改资源。
- 请求参数：GET请求的数据会附在URL之后。POST会把请求的数据放在HTTP请求报文的 请求体 中。
- 安全性：POST的安全性要比GET的安全性高。因为GET请求提交的数据将明文出现在URL上，而且POST请求参数则被包装到请求体中，相对更安全。



## Q:RESTful API

- 每个URL代表一种网络资源。因为表示资源，所以URL要用名词。
- 通过API对应的方法对服务器资源进行操作，如：GET获取资源、POST新建或更新资源、PUT更新资源、DELETE删除资源。
- 返回数据格式使用JSON。
- 它的提出是为了方便前后端分离后的交互。



## Q:HTTP与HTTPS的区别？

- HTTP和HTTPS都属于应用层协议。TCP属于传输层，SSL属于会话层。
- HTTP 协议运行在TCP上，明文传输，客户端和服务端无法验证对方身份。HTTPS是身披SSL外壳的HTTP，运行在SSL上，SSL运行在TCP上，是添加了加密和验证机制的HTTP。
- **端口：** HTTP使用80端口，HTTPS使用443。
- **资源开销：** 和HTTP相比，HTTPS会因为加解密消耗更多的CPU和资源。https连接也要比http流程长。

![](./static/164e529309f0fa33)



## Q:HTTP和TCP的区别？

- TCP是传输层协议，保证数据的可靠性传输。
- HTTP是应用层协议，建立在TCP连接之上，主要规定客户端和服务端的数据传输格式。



## Q:http 1.0 1.1 2.0 区别?

#### HTTP 1.0

- 默认不支持长连接，长连接需要设置`Connection: keep-alive`指定。
- **缓存处理：** 使用`header`里的`If-Modified-Since,Expires`来做为缓存判断的标准。

#### HTTP 1.1

- 默认支持长连接，多个http请求可以复用一个TCP连接，但是同一时间只能对应一个http请求(http请求在一个Tcp中是串行的)。
- **Host头处理** 在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。
- 引入了更多的缓存控制策略例如`Entity tag`，`If-Unmodified-Since`, `If-Match`, `If-None-Match`等更多可供选择的缓存头来控制缓存策略。

#### HTTP 2.0 （与HTTP1.X的区别）

- 默认支持长连接。
- 多路复用，一个Tcp中多个http请求是并行的 (雪碧图、多域名散列等优化手段http/2中将变得多余)
- 使用`HAPCK`算法对header数据进行压缩，使数据体积变小，传输更快。
- 支持服务器端推送。
- 协议解析使用二级制格式，方便健壮。

#### HTTP 3.0 (QUIC)

- HTTP 3.0 使用基于UDP协议的QUIC协议来实现，弃用 了TCP。
  - TCP协议由需要连接；协议头较大；弱网环境表现较差。
- 连接建立的延时低，一次往返可以建立HTTPS连接。
- 没有队头阻塞的多路复用。
- 改进的拥塞控制，高效的重传确认机制。
- 切换网络保证连接，从4G切换到WIFI不用重建连接。



## Q:HTTPS握手的五个阶段(HTTPS建立连接的过程)

- 客户端向服务器发送支持的SSL/TSL的协议版本号，以及客户端支持的加密方法，和一个客户端生成的随机数。
- 服务器确认协议版本号和加密方法，向客户端发送一个由服务器生成的随机数，以及数字证书。
- 客户端验证证书是否有效，有效则从证书中取出公钥，生成一个随机数，然后用公钥加密这个随机数，发给服务器。
- 服务器用私钥解密，获取客户端发来的随机数。
- 服务端根据预定好的加密方法，使用前面生成的三个随机数，生成对话密钥，发送给客户端。用来加密接下来的对话过程。

#### Reference

[https 原理](https://app.yinxiang.com/shard/s43/nl/13675070/cff50944-5c13-4ef9-8ca8-95de0d79d10a)



## Q:浏览器同源策略？

- **同源是指：** 协议相同、域名相同、端口相同。

- **同源策略的目的：** 保证用户信息的安全，防止恶意的网站窃取数据。

- **受限制行为：** 
  - Cookie、LocalStorage 和 IndexDB 无法读取。
  - DOM 无法获得。
  -  AJAX 请求不能发送。



## Q:跨域(CORS)如何设置?

> CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

- 允许浏览器向跨源服务器，发出`XMLHttpRequest`请求，从而克服了AJAX只能同源使用的限制。

- 客户端
  - CORS请求默认不发送Cookie和HTTP认证信息。如果要把Cookie发到服务器，一方面要服务器同意，指定`Access-Control-Allow-Credentials : true `字段。
  - 客户端请求的时候`withCredentials = true`
- 服务端
  - **Access-Control-Allow-Origin：** 该字段是必须的。它的值要么是请求时`Origin`字段的值，要么是一个`*`，表示接受任意域名的请求。
  - 复杂请求 - **Access-Control-Request-Method：** 该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，如PUT、DELETE。
  - 复杂请求 - **Access-Control-Request-Headers：** 该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段，上例是`X-Custom-Header`。
  - 简单请求 - **Access-Control-Allow-Credentials：** 可选，是一个布尔值，表示是否允许发送Cookie。
  - 简单请求 - **Access-Control-Expose-Headers：** 可选，CORS请求时，`XMLHttpRequest`对象的`getResponseHeader()`方法只能拿到6个基本字段：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`。如果想拿到其他字段，就必须在`Access-Control-Expose-Headers`里面指定。

[跨域共享资源详解 - 阮一峰](https://app.yinxiang.com/shard/s43/nl/13675070/8b715095-80e0-4eda-bc31-3782ba140002)



## Q:什么是websocket？ 它的应用场景？

- websocket是网络传输协议，位于OSI的应用层。基于TCP长连接进行全双工通信(服务端可以主动推送信息)，能更好的节省服务器资源和带宽并达到实时通迅。
- Websocket是基于HTTP协议去做握手的操作，`Upgrade: websocket; Connection: Upgrade`。
- 应用在弹幕、实时聊天等场景。



## Q:OAuth2.0协议是什么?

> 微信登录会获取code，使用code加上appid和appsecret换取access_token。

- OAuth2.0是一种授权协议。让某个应用（极客时间APP）能够以安全地方式获取到用户的委派书(access token)，随后应用便可以使用该委派书，代表用户来访问用户的相关资源(向微信获取信息)。

#### 授权模式

- Authorization Code（授权码 code）：服务器与客户端配合使用。
- Implicit（隐式 token）：用于移动应用程序或 Web 应用程序（在用户设备上运行的应用程序）。
- Resource Owner Password Credentials（资源所有者密码凭证 password）：资源所有者和客户端之间具有高度信任时（例如，客户端是设备的操作系统的一部分，或者是一个高度特权应用程序），以及当其他授权许可类型（例如授权码）不可用时被使用。
- Client Credentials（客户端证书 client_credentials）：当客户端代表自己表演（客户端也是资源所有者）或者基于与授权服务器事先商定的授权请求对受保护资源的访问权限时，客户端凭据被用作为授权许可。

