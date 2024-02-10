# AsyncUdpConnection

**(Requires workerman >= 3.0.8)**

AsyncUdpConnection puede actuar como un cliente UDP para comunicarse con un servidor remoto UDP.

De hecho, UDP es sin conexión, pero para mayor facilidad, aquí se mantiene casi igual la convención de nombres y la interfaz que AsyncTcpConnection.

**Nota: A diferencia de AsyncTcpConnection, AsyncUdpConnection no admite las siguientes propiedades o métodos.**
1. No hay propiedad connection->id
2. No hay propiedad connection->worker
3. No hay propiedad connection->transport
4. No hay propiedad connection->maxSendBufferSize
5. No hay propiedad connection->defaultMaxSendBufferSize
6. No hay propiedad connection->maxPackageSize
7. No hay callback connection->onBufferFull
8. No hay callback connection->onBufferDrain
9. No hay callback connection->onError
10. No hay método connection->destroy()
11. No hay método connection->pauseRecv()
12. No hay método connection->resumeRecv()
13. No hay método connection->pipe()
14. No hay método connection->reconnect()

**Propiedades o métodos admitidos por AsyncUdpConnection**
1. Admite la propiedad connection->protocol
2. Admite el callback connection->onMessage
3. Admite el método connection->connect()
4. Admite el método connection->send()
5. Admite el método connection->getRemoteIp()
6. Admite el método connection->getRemotePort()
7. Admite el callback connection->onClose.
   Nota: Debido a que TCP se basa en la conexión, en general, cuando cualquiera de las partes llama a close para desconectar la conexión, ambas partes pueden activar el evento onClose. Pero UDP es sin conexión, llamar al método connection->close() solo activará el callback onClose local, no puede activar el callback onClose del extremo remoto.
