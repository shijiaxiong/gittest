## select的非阻塞收发和随机执行

> select 的 case必须是channel的收发操作。

- `select` 能在channel 上进行非阻塞的收发操作。
  - 当存在可以收发的 Channel 时，直接处理该 Channel 对应的 `case`。比如已经关闭的通道。
  - 当不存在可以收发的Channel，执行default中的语句。
- `select` 在遇到多个channel同时响应时，会随机执行一种情况。

