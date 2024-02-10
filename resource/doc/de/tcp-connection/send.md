# send
## Erklärung:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

Sendet Daten an den Client

## Parameter

``` $data ```

Die zu sendenden Daten. Wenn beim Initialisieren der Worker-Klasse ein Protokoll angegeben wurde, wird automatisch die encode-Methode des Protokolls aufgerufen, um die Daten gemäß dem Protokoll zu verpacken und an den Client zu senden.

``` $raw ```

Gibt an, ob Rohdaten gesendet werden sollen, d.h. ob die encode-Methode des Protokolls nicht aufgerufen werden soll. Standardmäßig false, was bedeutet, dass die encode-Methode des Protokolls automatisch aufgerufen wird.

## Rückgabewert

true: Die Daten wurden erfolgreich in den Sendepuffer der Betriebssystemebene dieses Verbindung geschrieben.

null: Die Daten wurden in den Anwendungsschicht-Sendepuffer dieser Verbindung geschrieben und warten darauf, in den Sendepuffer der Betriebssystemebene zu gelangen.

false: Der Sendevorgang ist fehlgeschlagen, möglicherweise weil die Client-Verbindung geschlossen wurde oder der Anwendungsschicht-Sendepuffer voll ist.

## Hinweis
Wenn send true zurückgibt, bedeutet dies lediglich, dass die Daten erfolgreich in den Sendepuffer des Betriebssystems dieser Verbindung geschrieben wurden, was aber nicht bedeutet, dass die Daten an den Empfangspuffer des Gegenstücks erfolgreich gesendet wurden. Es bedeutet auch nicht, dass die Anwendungssoftware des Gegenstücks bereits Daten aus dem lokalen Sendepuffer des Sockets gelesen hat. **Aber selbst in diesem Fall können die Daten, solange send nicht false zurückgibt und das Netzwerk nicht getrennt ist und der Client normal empfängt, im Wesentlichen als zu 100% beim Gegenüber angekommen angesehen werden.**

Da die Daten im Sendepuffer des Sockets vom Betriebssystem asynchron an das Gegenstück gesendet werden, bietet das Betriebssystem keine entsprechende Bestätigungsmechanismen für die Anwendungsebene. Daher kann die Anwendungsebene nicht wissen, wann die Daten im Sendepuffer des Sockets zu senden beginnen, und die Anwendungsebene kann auch nicht wissen, ob die Daten im Sendepuffer des Sockets erfolgreich gesendet wurden. Aus diesen Gründen kann Workerman keine direkte Nachrichtenbestätigungsschnittstelle bereitstellen.

Wenn es notwendig ist, sicherzustellen, dass jeder Client jede Nachricht erhält, kann ein Bestätigungsmechanismus auf Anwendungsebene implementiert werden. Der Bestätigungsmechanismus kann je nach Geschäftsanforderungen variieren, und selbst für denselben Geschäftsbestätigungsmechanismus können verschiedene Methoden verwendet werden.

In einem Chat-System könnte beispielsweise ein solcher Bestätigungsmechanismus implementiert werden, bei dem jede Nachricht in der Datenbank gespeichert wird und jede Nachricht ein Feld hat, das angibt, ob sie gelesen wurde oder nicht. Wenn der Client eine Nachricht empfängt, sendet er eine Bestätigung an den Server, und der Server markiert dann die entsprechende Nachricht als gelesen. When the client connects to the server (usually when the user logs in or reconnects after a disconnection), it checks the database for unread messages and sends them to the client. Similarly, when the client receives a message, it notifies the server that it has been read. This way, it ensures that each message is received by the other party. Of course, developers can also use their own confirmation logic.


## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Hier wird automatisch \Workerman\Protocols\Websocket::encode aufgerufen, um die Daten gemäß dem Websocket-Protokoll zu senden
    $connection->send("hello\n");
};
// Worker ausführen
Worker::runAll();
```
