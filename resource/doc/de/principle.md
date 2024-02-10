# Prinzip

### Worker-Erklärung
Worker ist der grundlegendste Container in WorkerMan. Mehrere Worker können gestartet werden, um Ports zu überwachen und über ein bestimmtes Protokoll zu kommunizieren, ähnlich wie bei nginx, das einen bestimmten Port überwacht. Jeder Worker-Prozess arbeitet unabhängig und verwendet Epoll (erfordert die Installation der Event-Erweiterung) und nicht blockierende E/A. Jeder Worker-Prozess kann Tausende von Client-Verbindungen akzeptieren und die von diesen Verbindungen gesendeten Daten verarbeiten. Der Hauptprozess ist dafür verantwortlich, die Stabilität zu gewährleisten, überwacht jedoch nur die Kindprozesse, empfängt keine Daten und führt keine Geschäftslogik aus.

### Beziehung zwischen Client und Worker-Prozess
![workerman master woker模型](images/Worker.png)

### Beziehung zwischen Hauptprozess und Kindprozessen
![workerman master woker模型](images/Worker2.png)

**Merkmale:**

Aus dem Diagramm geht hervor, dass jeder Worker seine eigenen Client-Verbindungen verwaltet, um die Echtzeitkommunikation zwischen Client und Server problemlos zu realisieren. Auf der Grundlage dieses Modells können wir grundlegende Entwicklungsanforderungen wie HTTP-Server, RPC-Server, Echtzeit-Datenberichterstattung für intelligente Hardware, serverseitige Datenpushes, Spielserver, WeChat-Mini-Programm-Back-Ends usw. problemlos umsetzen.
