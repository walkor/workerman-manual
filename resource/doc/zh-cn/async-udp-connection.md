# AsyncUdpConnection

**(要求workerman>=3.0.8)**

AsyncUdpConnection可以作为udp客户端与远程udp服务端进行通讯。

其实udp是无连接的，但是为了易用性，这里与AsyncTcpConnection命名规则和接口保持基本一致。

**注意：与AsyncTcpConnection不同，AsyncUdpConnection不支持以下属性或者方法。**
1. 没有connection->id属性
2. 没有connection->worker属性
3. 没有connection->transport属性
4. 没有connection->maxSendBufferSize属性
5. 没有connection->defaultMaxSendBufferSize属性
6. 没有connection->maxPackageSize属性
7. 没有connection->onBufferFull回调
8. 没有connection->onBufferDrain回调
9. 没有connection->onError回调
10. 没有connection->destroy()接口
11. 没有connection->pauseRecv()接口
12. 没有connection->resumeRecv()接口
13. 没有connection->pipe()接口
14. 没有connection->reconnect()接口

**AsyncUdpConnection支持的属性或者方法**
1.支持connection->protocol属性
2.支持connection->onMessage回调
3.支持connection->connect()方法
4.支持connection->send()方法
5.支持connection->getRemoteIp()方法
6.支持connection->getRemotePort()方法
7.支持connection->onClose回调。
注意：因为tcp是基于连接的，一般情况下，当任何一方调用close断开连接时双方都能触发onClose。但是udp是无连接的，调用connection->close()方法只能触发本地的onClose回调，无法触发对端的onClose回调。

