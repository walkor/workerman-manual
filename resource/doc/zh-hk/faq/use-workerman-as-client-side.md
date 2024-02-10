# workerman可以作為客戶端接收處理來自遠程服務端的數據嗎？

可以利用AsyncTcpConnection發起異步連接，讓workerman作為客戶端與服務端交互。

例如下面的例子

1、[workerman作為websocket客戶端](as-wss-client.md)

2、[workerman作為mysql代理](../async-tcp-connection/connect.md)

3、[workerman作為http客戶端](../async-tcp-connection/construct.md)

4、[workerman作為http代理](https://github.com/walkor/php-http-proxy)

5、[workerman作為socks5代理](https://github.com/walkor/php-socks5)
