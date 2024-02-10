# onClose
## Erklärung:
```php
callback Worker::$onClose
```

Ein Rückruffunktion, die aufgerufen wird, wenn die Verbindung des Clients zu Workerman getrennt wird. Unabhängig davon, wie die Verbindung getrennt wird, wird ```onClose``` ausgelöst. Jede Verbindung löst nur einmal ```onClose``` aus.

Hinweis: Wenn die Gegenstelle die Verbindung unter extremen Bedingungen wie Netzwerkunterbrechung oder Stromausfall trennt, kann Workerman aufgrund der Unfähigkeit, das TCP-FIN-Paket rechtzeitig an Workerman zu senden, nicht erkennen, dass die Verbindung getrennt wurde, und kann daher nicht rechtzeitig ```onClose``` auslösen. In diesem Fall muss das Problem durch das Herzbeat auf Anwendungsebene gelöst werden. Sie können die Implementierung des Verbindungs-Herzschlags in Workerman [hier](../faq/heartbeat.md) nachsehen. Wenn Sie das GatewayWorker-Framework verwenden, können Sie einfach den Herzschlagmechanismus des GatewayWorker-Frameworks verwenden, siehe [hier](https://doc2.workerman.net/heartbeat.html). Bei der Verwendung von UDP wird keine onConnect-Callback oder onClose-Callback ausgelöst, da UDP verbindungslos ist.

## Parameter der Rückruffunktion
 ``` $connection ```

Das Verbindungsobjekt, d.h. die Instanz [TcpConnection](../tcp-connection.md), die für die Verwaltung der Clientverbindung verwendet wird, z.B. [Daten senden](../tcp-connection/send.md), [Verbindung schließen](../tcp-connection/close.md) usw.

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "Verbindung geschlossen\n";
};
// Worker ausführen
Worker::runAll();
```

Hinweis: Neben der Verwendung anonymer Funktionen als Rückruffunktion können auch andere Rückrufmethoden verwendet werden, siehe [hier](../faq/callback_methods.md).
