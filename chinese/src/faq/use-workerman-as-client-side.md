# workerman可以作为客户端接收处理来自远程服务端的数据么？


可以利用AsyncTcpConnection发起异步连接，让workerman作为客户端与服务端交互。

例如下面的例子

1、[workerman作为websocket客户端](/appendices/about-ws.html)

2、[workerman作为mysql代理](/worker-development/connect.html)

3、[workerman作为http客户端](/worker-development/__construct.html)

4、[workerman作为http代理](https://github.com/walkor/php-http-proxy)

5、[workerman作为socks5代理](https://github.com/walkor/php-socks5)

