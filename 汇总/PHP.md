## Q:Nginx 与 PHP-FPM的通讯方式？

- nginx 与 php-fpm 的通信有 tcp socket 和 unix socket 两种方式。

#### tcp socket

- tcp socket可以跨服务器。
- 可以更好的保障通讯的正确性和完整性。

#### unix socket

- 同一主机下的进程间通讯，效率比较高。不需要经过网络协议栈，只需要把数据从一个进程拷贝到另一个进程。
- unix socket 高并发时不稳定，连接数爆发时，会产生大量的长时缓存，在没有面向连接协议的支撑下，大数据包可能会直接出错不返回异常。
- 同一台服务器上运行的 nginx 和 php-fpm，且并发量不高（不超过1000），选择unix socket，以提高 nginx 和 php-fpm 的通信效率。

