# AsyncUdpConnection

**(Requires workerman>=3.0.8)**

AsyncUdpConnection pode ser usado como um cliente UDP para se comunicar com um servidor remoto UDP.

Na verdade, o UDP é sem conexão, mas para facilitar o uso, aqui mantenho as convenções de nomeação e interfaces semelhantes às do AsyncTcpConnection.

**Nota: Ao contrário do AsyncTcpConnection, o AsyncUdpConnection não suporta as seguintes propriedades ou métodos.**
1. Não possui a propriedade connection->id
2. Não possui a propriedade connection->worker
3. Não possui a propriedade connection->transport
4. Não possui a propriedade connection->maxSendBufferSize
5. Não possui a propriedade connection->defaultMaxSendBufferSize
6. Não possui a propriedade connection->maxPackageSize
7. Não possui o retorno de chamada connection->onBufferFull
8. Não possui o retorno de chamada connection->onBufferDrain
9. Não possui o retorno de chamada connection->onError
10. Não possui a interface connection->destroy()
11. Não possui a interface connection->pauseRecv()
12. Não possui a interface connection->resumeRecv()
13. Não possui a interface connection->pipe()
14. Não possui a interface connection->reconnect()

**Propriedades e métodos suportados por AsyncUdpConnection**
1. Suporta a propriedade connection->protocol
2. Suporta o retorno de chamada connection->onMessage
3. Suporta o método connection->connect()
4. Suporta o método connection->send()
5. Suporta o método connection->getRemoteIp()
6. Suporta o método connection->getRemotePort()
7. Suporta o retorno de chamada connection->onClose.
Nota: Como o TCP é baseado em conexão, geralmente, quando qualquer uma das partes chama o close para desconectar, ambas as partes podem acionar o onClose. Mas o UDP é sem conexão, chamar o método connection->close() só pode acionar o retorno de chamada onClose local, não pode acionar o retorno de chamada onClose do lado remoto.
