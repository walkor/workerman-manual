# Glatter Neustartprinzip
## Was ist ein glatter Neustart?

Ein glatter Neustart unterscheidet sich von einem normalen Neustart, da er es ermöglicht, den Dienst (in der Regel für Kurzverbindungen) neu zu starten, um das PHP-Programm neu zu laden und die Aktualisierung des Geschäftscode abzuschließen, ohne die Benutzer zu beeinträchtigen.

Ein glatter Neustart wird in der Regel während des Geschäfts-Updates oder Versionseinführungsprozesses angewendet, um vorübergehende Ausfallzeiten zu vermeiden, die durch das Neustarten des Dienstes aufgrund der Codebereitstellung verursacht werden.

> **Anmerkung**
> Windows-Systeme unterstützen kein "Reload".

> **Anmerkung**
> Bei Langzeitverbindungen (z. B. WebSocket) werden die Verbindungen beim vollständigen Neustart getrennt. Die Lösung besteht darin, eine Architektur ähnlich [gatewayWorker](https://www.workerman.net/doc/gateway-worker) zu verwenden, bei der eine Gruppe von Prozessen speziell zur Aufrechterhaltung der Verbindungen dient und deren [reloadable](../worker/reloadable.md)-Eigenschaft auf false gesetzt ist. Die Geschäftslogik startet eine separate Gruppe von Arbeitsprozessen, die miteinander und mit den Gateway-Prozessen über TCP kommunizieren. Wenn eine Aktualisierung des Geschäfts erforderlich ist, werden nur die Arbeitsprozesse neu gestartet.

## Einschränkungen
**Hinweis: Nur die nach dem on{...}-Rückruf geladenen Dateien werden nach einem glatten Neustart automatisch aktualisiert. Dateien, die direkt vom Startskript geladen werden oder in den Code eingefügt sind, werden beim Neustart nicht automatisch aktualisiert.**

#### Nachfolgender Code wird nach einem Neustart nicht aktualisiert
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // Der eingefügte Code unterstützt kein Hot-Update
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/dein/pfad/MessageHandler.php'; // Von Startskript direkt geladene Dateien unterstützen kein Hot-Update
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // Angenommen, die Klasse MessageHandler enthält eine Methode namens onMessage
```


#### Nachfolgender Code wird nach einem Neustart automatisch aktualisiert
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart wird nach dem Start des Prozesses aufgerufen
    require_once __DIR__ . '/dein/pfad/MessageHandler.php'; // Dateien, die nach dem Start des Prozesses geladen werden, unterstützen das Hot-Update
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
Nach Änderungen in der Datei MessageHandler.php wird durch Ausführen von `php start.php reload` die Datei in den Speicher neu geladen, um die Aktualisierung der Geschäftslogik zu erreichen.


> **Hinweis**
> Im obigen Code wurde für die Demonstration der `require_once`-Anweisung verwendet. Wenn Ihr Projekt das automatische Laden nach PSR-4 unterstützt, ist keine Verwendung der `require_once`-Anweisung erforderlich.

## Glatter Neustartprinzip

WorkerMan besteht aus einem Hauptprozess und den untergeordneten Prozessen. Der Hauptprozess überwacht die untergeordneten Prozesse, während die untergeordneten Prozesse die Verbindungen von Clients akzeptieren, auf eingehende Anforderungsdaten reagieren und diese entsprechend verarbeiten und an die Clients zurücksenden. Bei Aktualisierung des Geschäftscodes müssen wir nur die untergeordneten Prozesse aktualisieren, um den Code zu aktualisieren.

Wenn der WorkerMan-Hauptprozess das Signal für einen glatten Neustart empfängt, sendet der Hauptprozess ein sicheres Beenden-Signal an einen der untergeordneten Prozesse (um sicherzustellen, dass der entsprechende Prozess die aktuelle Anfrage erst abschließt, bevor er beendet wird). Nachdem dieser Prozess beendet ist, erstellt der Hauptprozess einen neuen untergeordneten Prozess (der den neuen PHP-Code lädt). Anschließend sendet der Hauptprozess erneut ein Stoppsignal an einen anderen alten Prozess, und auf diese Weise wird nach und nach ein Neustart-Prozess durchgeführt, bis alle alten Prozesse ersetzt sind.

Wie zu sehen ist, bedeutet ein glatter Neustart im Grunde genommen, dass die alten Geschäftsprozesse nach und nach beendet und neue Prozesse erstellt werden. Um die Benutzer bei einem glatten Neustart nicht zu beeinträchtigen, ist es wichtig, dass keine benutzerbezogenen Statusinformationen im Prozess gespeichert werden. Idealerweise sollte der Geschäftsprozess zustandslos sein, um einen Informationsverlust aufgrund des Prozessendes zu vermeiden.
