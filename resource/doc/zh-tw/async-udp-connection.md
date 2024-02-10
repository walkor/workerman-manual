# AsyncUdpConnection

**(要求workerman>=3.0.8)**

AsyncUdpConnection可以作為UDP客戶端與遠端UDP服務端進行通訊。

其實UDP是無連接的，但為了易用性，這裡與AsyncTcpConnection命名規則和接口保持基本一致。

**注意：與AsyncTcpConnection不同，AsyncUdpConnection不支援以下屬性或方法。**
1. 沒有connection->id屬性
2. 沒有connection->worker屬性
3. 沒有connection->transport屬性
4. 沒有connection->maxSendBufferSize屬性
5. 沒有connection->defaultMaxSendBufferSize屬性
6. 沒有connection->maxPackageSize屬性
7. 沒有connection->onBufferFull回調
8. 沒有connection->onBufferDrain回調
9. 沒有connection->onError回調
10. 沒有connection->destroy()接口
11. 沒有connection->pauseRecv()接口
12. 沒有connection->resumeRecv()接口
13. 沒有connection->pipe()接口
14. 沒有connection->reconnect()接口

**AsyncUdpConnection支援的屬性或方法**
1.支援connection->protocol屬性
2.支援connection->onMessage回調
3.支援connection->connect()方法
4.支援connection->send()方法
5.支援connection->getRemoteIp()方法
6.支援connection->getRemotePort()方法
7.支援connection->onClose回調。
注意：因為TCP是基於連接的，一般情況下，當任何一方調用close斷開連接時雙方都能觸發onClose。但是UDP是無連接的，調用connection->close()方法只能觸發本地的onClose回調，無法觸發對端的onClose回調。
