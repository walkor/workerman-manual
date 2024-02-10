# Einleitung

**Workerman, ein leistungsstarker PHP-Anwendungscontainer**

## Was ist Workerman?
Workerman ist ein Open-Source, leistungsstarker PHP-Anwendungscontainer, der rein in PHP entwickelt wurde.

Workerman ist keine redundante Erfindung, es ist kein MVC-Framework, sondern ein Framework, das auf einer tieferen und allgemeineren Ebene operiert. Mit Workerman können Sie TCP-Proxys, VPN-Proxys, Spiele-Server, E-Mail-Server, FTP-Server oder sogar eine PHP-Version von Redis, eine PHP-Version von einer Datenbank, eine PHP-Version von Nginx oder eine PHP-Version von PHP-FPM entwickeln. Workerman kann als eine Innovation im PHP-Bereich betrachtet werden, da es Entwicklern die Möglichkeit gibt, sich komplett von der Einschränkung zu befreien, dass PHP nur für das Web genutzt werden kann.

Tatsächlich ähnelt Workerman einer PHP-Version von Nginx, deren Kern auch auf Multiprozess, Epoll und nicht-blockierende IO basiert. Jeder Workerman-Prozess kann über zehntausende gleichzeitige Verbindungen aufrechterhalten. Da es im Speicher verbleibt und nicht auf Apache, Nginx oder PHP-FPM angewiesen ist, bietet es eine extrem hohe Leistung. Workerman unterstützt TCP, UDP, UNIXSOCKET, Langzeitverbindungen, sowie Kommunikationsprotokolle wie Websockets, HTTP, WSS und HTTPS, sowie verschiedene benutzerdefinierte Protokolle. Es verfügt über Timer, asynchrone Socket-Clients, asynchrones Redis, asynchrones HTTP, asynchrone Nachrichtenwarteschlangen und viele andere High-Performance-Komponenten.

## Einige Anwendungsrichtungen von Workerman
Im Gegensatz zu traditionellen MVC-Frameworks kann Workerman nicht nur für die Webentwicklung genutzt werden, sondern bietet auch breitere Anwendungsbereiche wie etwa Instant Messaging, das Internet der Dinge, Spiele, Service-Governance, andere Server oder Middleware. Dies erweitert zweifellos den Horizont der PHP-Entwickler. Derzeit gibt es einen Mangel an PHP-Entwicklern in diesen Bereichen. Wenn Sie im Bereich PHP technologische Vorteile erlangen möchten, sich nicht nur mit den täglichen CRUD-Arbeiten zufriedengeben oder in die Richtung eines Architekten oder technischen Experten entwickeln möchten, ist es definitiv sinnvoll, Workerman zu erlernen. Die Entwickler werden nicht nur empfohlen, es zu beherrschen, sondern auch, auf Basis von Workerman ihre eigenen Open-Source-Projekte zu entwickeln, um ihre Fähigkeiten zu verbessern und ihren Einfluss zu steigern, wie zum Beispiel [Beanbun Multiprozess-Webcrawler-Framework](https://github.com/kiddyuchina/Beanbun), das kürzlich online gegangen ist und bereits viele positive Bewertungen erhalten hat.

Einige Anwendungsrichtungen von Workerman umfassen:

1. Instant Messaging: z. B. Web-Instant-Messaging, sofortige Nachrichtenübermittlung, WeChat Mini-Programme, Push-Benachrichtigungen für mobile Apps und PC-Software ([Beispiel Workerman-Chatroom](https://www.workerman.net/workerman-chat), [Web-Push-Benachrichtigung](https://www.workerman.net/web-sender), [Toad Chatroom](https://www.workerman.net/workerman-todpole))

2. IoT: z. B. Kommunikation mit Druckern, Mikrocontrollern, Smartwatches, Smart-Home-Geräten, Shared Bikes, usw. [Beispiele von Kunden wie Yilianyun, Yiboshi](https://www.workerman.net)

3. Game Server: z. B. Kartenspiele, MMORPG-Spiele ([Beispiel Browserquest-PHP](https://www.workerman.net/browserquest))

4. HTTP-Dienste: z. B. Hochleistungs-HTTP-Schnittstellen, High-Performance-Websites. Wenn Sie HTTP-Dienste oder -Websites erstellen möchten, wird dringend die Verwendung von [webman](https://github.com/walkor/webman) empfohlen.

5. SOA-Dienste: Workerman kann bestehende Geschäftsfunktionen in Serviceeinheiten integrieren und diese als Dienste mit einer einzigen Schnittstelle nach außen anbieten, um ein lose gekoppeltes, wartbares, hochverfügbares und skalierbares System zu erzielen. ([Beispiel workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc), [workerman-thrift](https://github.com/walkor/workerman-thrift))

6. Andere Server-Software: z. B. [GatewayWorker](https://www.workerman.net/doc/gateway-worker), [PHPSocket.IO](https://www.workerman.net/phpsocket_io), [HTTP-Proxy](https://github.com/walkor/php-http-proxy), [SOCKS5-Proxy](https://github.com/walkor/php-socks5), verteilte Kommunikationskomponenten ([Channel](https://github.com/walkor/Channel)), verteilte Variablenkomponenten ([GlobalData](https://github.com/walkor/GlobalData)), Warteschlangen ([workerman-queue](https://github.com/walkor/workerman-queue)), DNS-Server, Webserver, CDN-Server, FTP-Server, usw.

7. Komponenten: z. B. asynchrones Redis, asynchroner HTTP-Client, MQTT-Client für das Internet der Dinge, Nachrichtenwarteschlange ([workerman/redis-queue](components/workerman-redis-queue.md), [workerman/stomp](components/workerman-stomp.md), [workerman/rabbitmq](components/workerman-rabbitmq.md)), Dateiüberwachungskomponenten, sowie viele von Drittanbietern entwickelte Komponenten-Frameworks, usw.

Es ist offensichtlich, dass traditionelle MVC-Frameworks nicht in der Lage sind, die oben genannten Funktionalitäten zu implementieren, was zur Entstehung von Workerman geführt hat.

## Die Philosophie von Workerman
Einfach, stabil, leistungsstark, verteilbar.

### **Einfach**
Klein ist schön, der Kern von Workerman ist extrem einfach, besteht nur aus einigen PHP-Dateien und verfügt nur über ein paar Schnittstellen, daher ist das Erlernen sehr einfach. Alle anderen Funktionen werden über Komponenten erweitert.

Workerman verfügt über eine vollständige Dokumentation, eine autorisierte Website, eine aktive Community, mehrere Tausend Mitglieder-QQ-Gruppen, zahlreiche High-Performance-Komponenten und unzählige Beispiele, was die Verwendung für Entwickler erleichtert.

### **Stabil**
Workerman ist seit einigen Jahren Open Source und wird von vielen börsennotierten Unternehmen in großem Umfang eingesetzt; es ist super stabil. Einige Dienste wurden seit mehr als 2 Jahren nicht neu gestartet und laufen immer noch reibungslos. Es gibt keine Core-Dumps, kein Memory-Leak, keine Bugs.

### **Leistungsstark**
Aufgrund des dauerhaften Speicherverbleibs und der Unabhängigkeit von Apache/Nginx/PHP-FPM, ohne Kommunikationsaufwand zwischen Containern und PHP und ohne Initialisierung und Bereinigung bei jeder Anfrage, bietet Workerman eine extrem hohe Leistung. Im Vergleich zu herkömmlichen MVC-Frameworks ist die Leistung um das Vielfache höher, unter PHP7 erreicht es sogar QPS-Werte, die höher sind als die von separaten Nginx-Servers unter Ab-Drucktests.

### **Verteilbar**
Es ist heutzutage nicht mehr die Zeit für Einzelkämpfer. Selbst wenn die Leistung eines einzelnen Servers beeindruckend ist, gibt es doch eine Grenze. Die richtige Methode ist ein verteiltes Multi-Server-Setup. Workerman bietet direkt eine verteilte Langzeitverbindungskommunikationslösung [GatewayWorker-Framework](https://doc2.workerman.net) an. Durch einfache Konfiguration und Starten von zusätzlichen Servern wird die Systemlast verdoppelt, ohne dass Änderungen am Geschäftscode erforderlich sind. Wenn Sie eine TCP-Langzeitverbindungsanwendung entwickeln möchten, wird dringend empfohlen, [GatewayWorker](https://doc2.workerman.net) zu verwenden. Es ist eine Erweiterung von Workerman, die eine reichhaltigere Schnittstelle und leistungsstarke verteilte Verarbeitungsfähigkeiten für Langzeitverbindungsanwendungen bietet.

## Geltungsbereich dieses Handbuchs
Workerman 3.x - 4.x Versionen

## Für Windows-Benutzer (Wichtig)
Workerman unterstützt sowohl Linux- als auch Windows-Systeme. Die Windows-Version von Workerman ist **nicht von zusätzlichen Erweiterungen abhängig**, es ist nur notwendig, die PHP-Umgebungsvariablen richtig zu konfigurieren. **Für die Installation und Hinweise zur Windows-Version von Workerman, beachten Sie bitte [die Anleitung für Windows-Benutzer](https://www.workerman.net/windows).**

## Client

Die Kommunikationsprotokolle von Workerman sind offen und anpassbar. Daher kann Workerman theoretisch mit Clients auf beliebigen Plattformen und unter Verwendung beliebiger Protokolle kommunizieren. Bei der Entwicklung von Client-Anwendungen können die Benutzer nach dem entsprechenden Kommunikationsprotokoll die Kommunikation mit dem Server realisieren.
