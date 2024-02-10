# AsyncUdpConnection

**(workerman>=3.0.8が必要です)**

AsyncUdpConnectionは、UDPクライアントとリモートUDPサーバー間で通信するためのものです。

実際にはUDPは非接続型ですが、使いやすさのために、ここではAsyncTcpConnectionの命名規則とインターフェースを基本的に一致させています。

**注意：AsyncTcpConnectionとは異なり、AsyncUdpConnectionは以下のプロパティやメソッドをサポートしていません。**
1. connection->idプロパティはありません
2. connection->workerプロパティはありません
3. connection->transportプロパティはありません
4. connection->maxSendBufferSizeプロパティはありません
5. connection->defaultMaxSendBufferSizeプロパティはありません
6. connection->maxPackageSizeプロパティはありません
7. connection->onBufferFullコールバックはありません
8. connection->onBufferDrainコールバックはありません
9. connection->onErrorコールバックはありません
10. destroy()インターフェースはありません
11. pauseRecv()インターフェースはありません
12. resumeRecv()インターフェースはありません
13. pipe()インターフェースはありません
14. reconnect()インターフェースはありません

**AsyncUdpConnectionでサポートされるプロパティやメソッド**
1. connection->protocolプロパティをサポートしています
2. connection->onMessageコールバックをサポートしています
3. connection->connect()メソッドをサポートしています
4. connection->send()メソッドをサポートしています
5. connection->getRemoteIp()メソッドをサポートしています
6. connection->getRemotePort()メソッドをサポートしています
7. connection->onCloseコールバックをサポートしています。
注意：TCPは接続に基づいているため、通常、接続を切断すると両方がonCloseをトリガーできます。しかし、UDPは非接続型なので、connection->close()メソッドを呼び出すとローカルのonCloseコールバックがトリガーされますが、対向のonCloseコールバックをトリガーすることはできません。
