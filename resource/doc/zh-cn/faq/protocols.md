# WorkerMan支持哪些协议

WorkerMan在接口上支持各种协议，只要符合```ConnectionInterface```接口即可（参见定制通讯协议章节）。

为了方便开发者，WorkerMan提供了HTTP协议、WebSocket协议以及非常简单的Text文本协议、可用于二进制传输的frame协议。开发者可以直接使用这些协议，不必再二次开发。如果这些协议都不满足需要，开发者可以参照定制协议章节实现自己的协议。

开发者也可以直接基于tcp或者udp协议。

协议使用示例
```php
// http协议
$worker1 = new Worker('http://0.0.0.0:1221');
// websocket协议
$worker2 = new Worker('websocket://0.0.0.0:1222');
// text文本协议（telnet协议）
$worker3 = new Worker('text://0.0.0.0:1223');
// frame文本协议（可用于二进制数传输）
$worker3 = new Worker('frame://0.0.0.0:1223');
// 直接基于tcp传输
$worker4 = new Worker('tcp://0.0.0.0:1224');
// 直接基于udp传输
$worker5 = new Worker('udp://0.0.0.0:1225');
```
