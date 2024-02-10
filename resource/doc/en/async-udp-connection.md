# AsyncUdpConnection

**(Requires workerman>=3.0.8)**

AsyncUdpConnection can be used as a udp client to communicate with remote udp server.

Actually, udp is connectionless, but for ease of use, the naming rules and interfaces are kept basically the same as AsyncTcpConnection.

**Note: Unlike AsyncTcpConnection, AsyncUdpConnection does not support the following properties or methods.**
1. No connection->id property
2. No connection->worker property
3. No connection->transport property
4. No connection->maxSendBufferSize property
5. No connection->defaultMaxSendBufferSize property
6. No connection->maxPackageSize property
7. No connection->onBufferFull callback
8. No connection->onBufferDrain callback
9. No connection->onError callback
10. No connection->destroy() method
11. No connection->pauseRecv() method
12. No connection->resumeRecv() method
13. No connection->pipe() method
14. No connection->reconnect() method

**Properties or methods supported by AsyncUdpConnection**
1. Supports connection->protocol property
2. Supports connection->onMessage callback
3. Supports connection->connect() method
4. Supports connection->send() method
5. Supports connection->getRemoteIp() method
6. Supports connection->getRemotePort() method
7. Supports connection->onClose callback.
Note: Because tcp is connection-based, generally, when either side calls close to disconnect the connection, both sides can trigger onClose. But udp is connectionless, calling the connection->close() method can only trigger the local onClose callback and cannot trigger the onClose callback on the other end.
