# AsyncUdpConnection

**(workerman >= 3.0.8 gerektirir)**

AsyncUdpConnection, uzak udp sunucusu ile iletişim kurmak için udp istemcisi olarak kullanılabilir.

Aslında udp bağlantısızdır, ancak kolaylık sağlamak için burada AsyncTcpConnection adlandırma kuralı ve arabirimine temelde uyumlu tutulmuştur.

**Not: AsyncTcpConnection ile farklı olarak, AsyncUdpConnection aşağıdaki özellikleri veya yöntemleri desteklemez.**
1. connection->id özelliği yoktur
2. connection->worker özelliği yoktur
3. connection->transport özelliği yoktur
4. connection->maxSendBufferSize özelliği yoktur
5. connection->defaultMaxSendBufferSize özelliği yoktur
6. connection->maxPackageSize özelliği yoktur
7. connection->onBufferFull geri çağrısı yoktur
8. connection->onBufferDrain geri çağrısı yoktur
9. connection->onError geri çağrısı yoktur
10. connection->destroy() arabirimi yoktur
11. connection->pauseRecv() arabirimi yoktur
12. connection->resumeRecv() arabirimi yoktur
13. connection->pipe() arabirimi yoktur
14. connection->reconnect() arabirimi yoktur

**AsyncUdpConnection tarafından desteklenen özellikler veya yöntemler**
1. connection->protocol özelliği desteklenir
2. connection->onMessage geri çağrısı desteklenir
3. connection->connect() yöntemi desteklenir
4. connection->send() yöntemi desteklenir
5. connection->getRemoteIp() yöntemi desteklenir
6. connection->getRemotePort() yöntemi desteklenir
7. connection->onClose geri çağrısı desteklenir.
Not: TCP bağlantısına dayalı olduğu için genellikle her iki tarafın da close çağrısı yaptığında onClose tetiklenebilir. Ancak UDP bağlantısızdır, connection->close() yöntemi yalnızca yerel onClose geri çağrısını tetikleyebilir, karşı taraftaki onClose geri çağrısını tetikleyemez.
