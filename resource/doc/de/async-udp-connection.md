# AsyncUdpConnection

**(Requires workerman>=3.0.8)**

AsyncUdpConnection ermöglicht die Kommunikation als UDP-Client mit entfernten UDP-Servern.

Obwohl UDP verbindungslos ist, wurde die Namenskonvention und die Schnittstelle hier aus Gründen der Benutzerfreundlichkeit im Wesentlichen gleich wie bei AsyncTcpConnection beibehalten.

**Hinweis: Im Gegensatz zu AsyncTcpConnection unterstützt AsyncUdpConnection nicht die folgenden Eigenschaften oder Methoden.**
1. Die Eigenschaft connection->id existiert nicht.
2. Die Eigenschaft connection->worker existiert nicht.
3. Die Eigenschaft connection->transport existiert nicht.
4. Die Eigenschaft connection->maxSendBufferSize existiert nicht.
5. Die Eigenschaft connection->defaultMaxSendBufferSize existiert nicht.
6. Die Eigenschaft connection->maxPackageSize existiert nicht.
7. Es gibt keinen Rückruf für connection->onBufferFull.
8. Es gibt keinen Rückruf für connection->onBufferDrain.
9. Es gibt keinen Rückruf für connection->onError.
10. Die Methode connection->destroy() existiert nicht.
11. Die Methode connection->pauseRecv() existiert nicht.
12. Die Methode connection->resumeRecv() existiert nicht.
13. Die Methode connection->pipe() existiert nicht.
14. Die Methode connection->reconnect() existiert nicht.

**Eigenschaften oder Methoden, die von AsyncUdpConnection unterstützt werden:**
1. Die Eigenschaft connection->protocol wird unterstützt.
2. Der Rückruf connection->onMessage wird unterstützt.
3. Die Methode connection->connect() wird unterstützt.
4. Die Methode connection->send() wird unterstützt.
5. Die Methode connection->getRemoteIp() wird unterstützt.
6. Die Methode connection->getRemotePort() wird unterstützt.
7. Der Rückruf connection->onClose wird unterstützt.
Hinweis: Da TCP auf Verbindungen basiert, wird in der Regel bei einem Aufruf von close durch eine der Parteien die Verbindung getrennt und es wird onClose beider Seiten ausgelöst. Bei UDP ist jedoch keine Verbindung vorhanden. Wenn die Methode connection->close() aufgerufen wird, wird nur der lokale onClose-Rückruf ausgelöst und der onClose-Rückruf auf der anderen Seite kann nicht ausgelöst werden.
