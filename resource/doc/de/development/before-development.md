# Entwicklungsvorbereitung

Um eine Anwendung mit WorkerMan zu entwickeln, müssen Sie die folgenden Inhalte verstehen:

## I. Unterschiede zwischen WorkerMan-Entwicklung und herkömmlicher PHP-Entwicklung

Abgesehen davon, dass Variablenfunktionen im Zusammenhang mit dem HTTP-Protokoll nicht direkt verwendet werden können, gibt es nicht viele Unterschiede zwischen der Entwicklung mit WorkerMan und der herkömmlichen PHP-Entwicklung.

### 1. Unterschiede in der Anwendungsprotokollierung
- Herkömmliche PHP-Entwicklung basiert im Allgemeinen auf dem HTTP-Anwendungsprotokoll, bei dem der Webserver die Protokollierung für Entwickler übernimmt
- WorkerMan unterstützt verschiedene Protokolle und hat derzeit HTTP, WebSocket und andere Protokolle integriert. WorkerMan empfiehlt Entwicklern, einfachere benutzerdefinierte Kommunikationsprotokolle zu verwenden

### 2. Unterschiede im Anfragezyklus
- PHP in Webanwendungen gibt nach einer Anfrage alle Variablen und Ressourcen frei
- Anwendungen, die mit WorkerMan entwickelt wurden, bleiben nach dem ersten Laden und Parsen im Speicher und ermöglichen es Klassendefinitionen, globalen Objekten und statischen Mitgliedern, wiederverwendet zu werden

### 3. Vermeidung von Mehrfachdefinitionen von Klassen und Konstanten
- Da WorkerMan die kompilierten PHP-Dateien im Cache speichert, sollte vermieden werden, dass dieselben Klassen oder Konstantendefinitionen mehrmals eingebunden werden. Die Verwendung von require_once/include_once wird empfohlen.

### 4. Freigabe von Verbindungsmitteln im Singleton-Modus
- Da WorkerMan die globalen Objekte und statischen Mitglieder beim Beenden der Anfrage nicht freigibt, werden Verbindungen in Singletons wie Datenbankinstanzen (einschließlich einer Datenbank-Socket-Verbindung) oft im statischen Mitglied der Datenbank gespeichert. Dies ermöglicht es WorkerMan, diese Datenbank-Socket-Verbindung während der gesamten Lebensdauer des Prozesses wiederzuverwenden.

### 5. Vermeidung von exit- und die-Anweisungen
- WorkerMan läuft im PHP-CLI-Modus. Wenn exit- oder die-Anweisungen aufgerufen werden, führt dies zum Beenden des aktuellen Prozesses, was selbst bei der sofortigen Erstellung eines identischen Prozesses möglicherweise Auswirkungen auf das Geschäft haben kann.

### 6. Neustart des Dienstes nach Codeänderungen
Da WorkerMan im Speicher verbleibt und die Definition von PHP-Klassen und -Funktionen beim ersten Laden im Speicher verbleibt und nicht erneut vom Datenträger geladen wird, müssen nach jeder Codeänderung der Geschäftslogik der Dienst neu gestartet werden, um wirksam zu werden.

## II. Grundlegende Konzepte, die Sie kennen müssen

### 1. TCP-Transportprotokoll
TCP ist ein verbindungsorientiertes, zuverlässiges, auf IP basierendes Transportprotokoll. Ein wichtiges Merkmal von TCP ist, dass es auf Datenflüsse basiert. Die Anfragen des Clients werden kontinuierlich an den Server gesendet, und die Daten, die der Server empfängt, sind möglicherweise keine vollständige Anfrage, sondern möglicherweise mehrere Anfragen hintereinander. Hierbei ist es erforderlich, in diesem kontinuierlichen Datenfluss die Grenzen jeder Anfrage zu unterscheiden. Die Anwendungsprotokolle dienen hauptsächlich dazu, Regeln für die Abgrenzung von Anfragedaten festzulegen und Datenstörungen zu vermeiden.

### 2. Anwendungsprotokoll
Ein Anwendungsprotokoll definiert, wie Anwendungsprogrammprozesse auf verschiedenen Endsystemen (Client, Server) Nachrichten austauschen. Beispiele für Anwendungsprotokolle sind HTTP, WebSocket usw. Ein einfaches Anwendungsprotokoll kann beispielsweise folgendermaßen aussehen: ```{"module":"user","action":"getInfo","uid":456}\n"```. Dieses Protokoll markiert das Ende der Anfrage mit ```"\n"``` (Beachten Sie, dass ```"\n"``` für einen Zeilenumbruch steht). Die Nachrichten sind in Form von Zeichenfolgen.

### 3. Kurzverbindungen
Kurzverbindungen bedeuten, dass beim Datenaustausch zwischen den beiden Parteien eine Verbindung hergestellt wird, die Verbindung nach Abschluss des Datenaustauschs jedoch getrennt wird, dh dass jede Verbindung nur für eine Geschäftstransaktion hergestellt wird. Web-HTTP-Dienste verwenden im Allgemeinen Kurzverbindungen.

### 4. Langverbindungen
Langverbindungen ermöglichen das kontinuierliche Versenden mehrerer Datenpakete über eine Verbindung. Beachten Sie, dass Langverbindungen ein [Herzschlag](../faq/heartbeat.md) benötigen, da die Verbindung aufgrund langer Inaktivität von Routennodes oder Firewalls getrennt werden kann.

### 5. Sanftes Neustarten
Ein herkömmlicher Neustart beinhaltet das Stoppen aller Prozesse und dann das Starten neuer Dienstprozesse. Während dieses Vorgangs gibt es eine kurze Zeit, in der keine Prozesse für Dienste verfügbar sind, was bei hoher Auslastung zu Anfragefehlern führen kann. Sanftes Neustarten beinhaltet nicht das gleichzeitige Beenden aller Prozesse, sondern das schrittweise Beenden und Ersetzen jedes alten Prozesses durch einen neuen Prozess, bis alle alten Prozesse ersetzt sind.
Sanftes Neustarten in WorkerMan kann mit dem Befehl ```php your_file.php reload``` durchgeführt werden, um Anwendungsaktualisierungen ohne Beeinträchtigung der Dienstqualität zu ermöglichen.

**Hinweis: Nur Dateien, die in den on{...}-Rückrufen geladen werden, werden nach einem sanften Neustart automatisch aktualisiert. Dateien, die direkt in das Startskript geladen werden oder deren Code festgeschrieben ist, werden beim Neustart nicht automatisch aktualisiert.**

## III. Unterscheidung zwischen Hauptprozessen und Unterprozessen
Es ist wichtig zu beachten, ob der Code in einem Hauptprozess oder einem Unterprozess ausgeführt wird. Im Allgemeinen wird der Code vor dem Aufruf von ```Worker::runAll();``` im Hauptprozess ausgeführt, und die in den onXXX-Rückrufen ausgeführten Codes gehören zu den Unterprozessen. Beachten Sie, dass der Code, der nach ```Worker::runAll();``` geschrieben wird, niemals ausgeführt wird.

Beispielsweise der folgende Code:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// im Hauptprozess ausgeführt
$tcp_worker = new Worker("tcp://0.0.0.0:2347");
// der Zuweisungsvorgang wird im Hauptprozess ausgeführt
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Dieser Teil wird im Unterprozess ausgeführt
    $connection->send('hello ' . $data);
};

Worker::runAll();
```

**Hinweis:** Initialisieren Sie keine Verbindungsressourcen wie Datenbanken, Memcache, Redis usw. im Hauptprozess, da die vom Hauptprozess initialisierten Verbindungen möglicherweise automatisch von den Unterprozessen übernommen werden (insbesondere bei Verwendung von Singletons), sodass alle Prozesse über dieselbe Verbindung verfügen. Die vom Server über diese Verbindung zurückgegebenen Daten können von mehreren Prozessen gelesen werden und führen zu Datenstörungen. Gleichermaßen wird jede Schließung der Verbindung durch einen Prozess (z. B. wenn der Hauptprozess beim Ausführen des Daemon-Modus beendet wird, was zur Schließung der Verbindung führt) dazu führen, dass alle Unterprozesse ihre Verbindung ebenfalls schließen, was zu unvorhersehbaren Fehlern führen kann, wie beispielsweise dem „MySQL Gone Away“-Fehler.

Es wird empfohlen, Verbindungsressourcen im onWorkerStart zu initialisieren.

