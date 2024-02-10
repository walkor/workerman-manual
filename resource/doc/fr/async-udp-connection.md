# AsyncUdpConnection

**(Requires workerman>=3.0.8)**

AsyncUdpConnection peut être utilisé comme un client udp pour communiquer avec un serveur distant udp.

En fait, l'udp est sans connexion, mais pour des raisons de facilité d'utilisation, ici, la règle de dénomination et l'interface sont maintenues de manière similaire à AsyncTcpConnection.

**Remarque : Contrairement à AsyncTcpConnection, AsyncUdpConnection ne prend pas en charge les propriétés ou méthodes suivantes.**
1. Pas de propriété connection->id
2. Pas de propriété connection->worker
3. Pas de propriété connection->transport
4. Pas de propriété connection->maxSendBufferSize
5. Pas de propriété connection->defaultMaxSendBufferSize
6. Pas de propriété connection->maxPackageSize
7. Pas de rappel onBufferFull pour la propriété connection
8. Pas de rappel onBufferDrain pour la propriété connection
9. Pas de rappel onError pour la propriété connection
10. Pas de méthode destroy() pour la propriété connection
11. Pas de méthode pauseRecv() pour la propriété connection
12. Pas de méthode resumeRecv() pour la propriété connection
13. Pas de méthode pipe() pour la propriété connection
14. Pas de méthode reconnect() pour la propriété connection

**Propriétés ou méthodes prises en charge par AsyncUdpConnection**
1. Prend en charge la propriété connection->protocol
2. Prend en charge le rappel onMessage pour la propriété connection
3. Prend en charge la méthode connect() pour la propriété connection
4. Prend en charge la méthode send() pour la propriété connection
5. Prend en charge la méthode getRemoteIp() pour la propriété connection
6. Prend en charge la méthode getRemotePort() pour la propriété connection
7. Prend en charge le rappel onClose pour la propriété connection.
Note : Comme le tcp est basé sur une connexion, en général, lorsque l'une ou l'autre des parties appelle close pour rompre la connexion, les deux parties peuvent déclencher onClose. Mais l'udp est sans connexion, en appelant la méthode connection->close(), seul le rappel onClose local peut être déclenché, et il est impossible de déclencher le rappel onClose distant.
