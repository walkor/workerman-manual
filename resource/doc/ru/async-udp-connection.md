# AsyncUdpConnection

**(требуется workerman>=3.0.8)**

AsyncUdpConnection может использоваться в качестве udp-клиента для связи с удаленным udp-сервером.

Фактически udp не имеет соединения, однако для удобства здесь используется та же система и интерфейс, что и у AsyncTcpConnection.

**Примечание: В отличие от AsyncTcpConnection, AsyncUdpConnection не поддерживает следующие свойства или методы.**
1. Нет свойства connection->id
2. Нет свойства connection->worker
3. Нет свойства connection->transport
4. Нет свойства connection->maxSendBufferSize
5. Нет свойства connection->defaultMaxSendBufferSize
6. Нет свойства connection->maxPackageSize
7. Нет обратного вызова connection->onBufferFull
8. Нет обратного вызова connection->onBufferDrain
9. Нет обратного вызова connection->onError
10. Нет метода destroy() интерфейса
11. Нет метода pauseRecv() интерфейса
12. Нет метода resumeRecv() интерфейса
13. Нет метода pipe() интерфейса
14. Нет метода reconnect() интерфейса

**Свойства или методы, поддерживаемые AsyncUdpConnection**
1. Поддерживается свойство connection->protocol
2. Поддерживается обратный вызов connection->onMessage
3. Поддерживается метод connection->connect()
4. Поддерживается метод connection->send()
5. Поддерживается метод connection->getRemoteIp()
6. Поддерживается метод connection->getRemotePort()
7. Поддерживается обратный вызов connection->onClose.
Примечание: поскольку tcp основан на соединении, в общем случае, когда одна из сторон вызывает close для разрыва соединения, обе стороны могут вызвать onClose. Однако udp не имеет соединения, поэтому вызов метода connection->close() может вызвать только локальный обратный вызов onClose, не вызывая обратного вызова onClose на удаленной стороне.
