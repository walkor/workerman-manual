# 非同步 UDP 連接

**(需要 workerman>=3.0.8)**

非同步 UDP 連接可用作 UDP 客戶端與遠端 UDP 服務端進行通訊。

事實上，UDP 是無連接的，但是為了易用性，在這裡與非同步 TCP 連接的命名規則和接口保持基本一致。

**注意：與非同步 TCP 連接不同，非同步 UDP 連接不支持以下屬性或方法。**
1. 沒有 connection->id 屬性
2. 沒有 connection->worker 屬性
3. 沒有 connection->transport 屬性
4. 沒有 connection->maxSendBufferSize 屬性
5. 沒有 connection->defaultMaxSendBufferSize 屬性
6. 沒有 connection->maxPackageSize 屬性
7. 沒有 connection->onBufferFull 回調
8. 沒有 connection->onBufferDrain 回調
9. 沒有 connection->onError 回調
10. 沒有 connection->destroy() 接口
11. 沒有 connection->pauseRecv() 接口
12. 沒有 connection->resumeRecv() 接口
13. 沒有 connection->pipe() 接口
14. 沒有 connection->reconnect() 接口

**非同步 UDP 連接支持的屬性或方法**
1. 支持 connection->protocol 屬性
2. 支持 connection->onMessage 回調
3. 支持 connection->connect() 方法
4. 支持 connection->send() 方法
5. 支持 connection->getRemoteIp() 方法
6. 支持 connection->getRemotePort() 方法
7. 支持 connection->onClose 回調。
**注意：因為 TCP 是基於連接的，一般情況下，當任何一方調用 close 斷開連接時，雙方都能觸發 onClose。但是 UDP 是無連接的，調用 connection->close() 方法只能觸發本地的 onClose 回調，無法觸發對端的 onClose 回調。**
