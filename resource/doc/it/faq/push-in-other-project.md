# Utilizzare WorkerMan in altri progetti per inviare dati ai clienti

**Domanda:**

Ho un normale progetto web e vorrei chiamare l'API di WorkerMan in questo progetto per inviare dati ai clienti.

**Risposta:**

**Basato su Workerman, si possono consultare i seguenti collegamenti**

- [Esempio di push del componente Channel](../components/channel-examples.md) (supporta multi-processi/cluster server, richiede il download del componente Channel)

- [Push basato su Worker](https://www.workerman.net/q/508) (single processo, il più semplice)

**Riferimento basato su webman**

- [Plugin push di webman](https://www.workerman.net/plugin/2)

**Riferimento basato su GatewayWorker**

- [Push attraverso GatewayWorker in altri progetti](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html) (supporta multi-processi/cluster server, supporta gruppi, multicast, e invio singolo)

**Riferimento basato su PHPSocket.IO**

- [Push messaggio web](https://www.workerman.net/web-sender) (predefinito single processo, basato su socket.io, la migliore compatibilità del browser)
