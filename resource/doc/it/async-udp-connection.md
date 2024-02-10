# AsyncUdpConnection

**(Richiesto workerman>=3.0.8)**

AsyncUdpConnection può essere utilizzato come client UDP per comunicare con un server UDP remoto.

In realtà, l'UDP è senza connessione, ma per facilità d'uso, qui viene mantenuta la stessa convenzione di denominazione e interfaccia di AsyncTcpConnection.

**Nota: A differenza di AsyncTcpConnection, AsyncUdpConnection non supporta le seguenti proprietà o metodi.**
1. Non ha la proprietà connection->id.
2. Non ha la proprietà connection->worker.
3. Non ha la proprietà connection->transport.
4. Non ha la proprietà connection->maxSendBufferSize.
5. Non ha la proprietà connection->defaultMaxSendBufferSize.
6. Non ha la proprietà connection->maxPackageSize.
7. Non ha il callback connection->onBufferFull.
8. Non ha il callback connection->onBufferDrain.
9. Non ha il callback connection->onError.
10. Non ha il metodo connection->destroy().
11. Non ha il metodo connection->pauseRecv().
12. Non ha il metodo connection->resumeRecv().
13. Non ha il metodo connection->pipe().
14. Non ha il metodo connection->reconnect().

**Proprietà e metodi supportati da AsyncUdpConnection**
1. Supporta la proprietà connection->protocol.
2. Supporta il callback connection->onMessage.
3. Supporta il metodo connection->connect().
4. Supporta il metodo connection->send().
5. Supporta il metodo connection->getRemoteIp().
6. Supporta il metodo connection->getRemotePort().
7. Supporta il callback connection->onClose.
Nota: Poiché TCP è basato su connessione, di solito quando una delle parti chiama close per interrompere la connessione, entrambe le parti possono attivare onClose. Tuttavia, poiché UDP è senza connessione, chiamare il metodo connection->close() può attivare solo il callback onClose locale, senza poter attivare il callback onClose della parte remota.
