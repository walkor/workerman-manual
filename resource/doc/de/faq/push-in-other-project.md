# Verwendung von WorkerMan, um Daten an Clients in anderen Projekten zu senden

**Frage:**

Ich habe ein normales Webprojekt und möchte WorkerMan-APIs in diesem Projekt aufrufen, um Daten an Clients zu senden.

**Antwort:**

**Verweise für die Verwendung von Workerman:**

- [Beispiel für Channel-Komponente-Push](../components/channel-examples.md) (Unterstützt Multiprozess/Server-Cluster, erfordert den Download der Channel-Komponente)

- [Push mit dem Worker](https://www.workerman.net/q/508) (Einzelprozess, am einfachsten)

**Verweise für die Verwendung von webman:**

- [webman Push-Plugin](https://www.workerman.net/plugin/2)

**Verweise für die Verwendung von GatewayWorker:**

- [Push über GatewayWorker in anderen Projekten](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html) (Unterstützt Multiprozess/Server-Cluster, unterstützt Gruppen, Gruppenübertragung und Einzelübertragung)

**Verweise für die Verwendung von PHPSocket.IO:**

- [Webnachrichten-Push](https://www.workerman.net/web-sender) (Standardmäßig Einzelprozess, basierend auf socket.io, beste Browserkompatibilität)
