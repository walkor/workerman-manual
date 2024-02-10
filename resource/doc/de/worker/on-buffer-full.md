# onBufferFull
## Erklärung:
```php
callback Worker::$onBufferFull
```

Jede Verbindung verfügt über einen separaten Anwendungsschicht-Sende-Puffer. Wenn die Geschwindigkeit, mit der der Client Daten empfängt, geringer ist als die Geschwindigkeit, mit der der Server Daten sendet, werden die Daten im Anwendungsschicht-Puffer zwischengespeichert. Wenn der Puffer voll ist, wird das onBufferFull-Rückrufereignis ausgelöst.

Der Puffer hat die maximale Größe [TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md) und den Standardwert von 1 MB. Sie können die Puffergröße für die aktuelle Verbindung dynamisch festlegen, beispielsweise:
```php
// Setzen Sie die maximale Sendepuffergröße für die aktuelle Verbindung in Byte
$verbindung->maxSendBufferSize = 102400;
```
Sie können auch [TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) verwenden, um die Standardpuffergröße für alle Verbindungen festzulegen, z. B.:
```php
use Workerman\Connection\TcpConnection;
// Setzen Sie die Standard-Größe für den Anwendungsschicht-Sende-Puffer für alle Verbindungen in Byte
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

Dieses Rückrufereignis kann **möglicherweise** unmittelbar nach dem Aufruf von Connection::send ausgelöst werden, z. B. wenn große Daten gesendet werden oder Daten schnell hintereinander an den Endpunkt gesendet werden. Aufgrund von Netzwerkproblemen und anderen Gründen können Daten in großen Mengen im Sende-Puffer der entsprechenden Verbindung angesammelt werden und führen zum Auslösen des Ereignisses, wenn die Grenze von ```TcpConnection::$maxSendBufferSize``` überschritten wird.

Wenn das onBufferFull-Ereignis eintritt, sollte der Entwickler Maßnahmen ergreifen, beispielsweise das Stoppen des Sendens von Daten an den Endpunkt und das Warten, bis die Daten im Sende-Puffer gesendet wurden (onBufferDrain-Ereignis).

Wenn der Aufruf von Connection::send(`$A`) zum Auslösen von onBufferFull führt, wird unabhängig von der Größe der aktuellen Daten `$A`, auch wenn sie größer als `TcpConnection::$maxSendBufferSize` ist, werden die zu sendenden Daten immer in den Sende-Puffer gelegt. Das heißt, die tatsächlich im Sende-Puffer abgelegten Daten können wesentlich größer sein als `TcpConnection::$maxSendBufferSize`. Wenn der Sende-Puffer bereits über Daten im Umfang von `TcpConnection::$maxSendBufferSize` verfügt und erneut Connection::send(`$B`) aufgerufen wird, werden die Daten von `$B` nicht in den Sende-Puffer gelegt, sondern verworfen und ein `onError`-Rückruf ausgelöst.

Zusammenfassend gilt: Solange der Sende-Puffer nicht voll ist, selbst wenn nur ein Byte Platz ist, wird der Aufruf von Connection::send(```$A```) die Daten `$A` sicher in den Sende-Puffer legen. Wenn der Sende-Puffer die Grenze von `TcpConnection::$maxSendBufferSize` überschreitet, wird das onBufferFull-Ereignis ausgelöst.

## Parameter der Rückruffunktion
``` $connection ```

Verbindungsobjekt, d. h. eine [TcpConnection-Instanz](../tcp-connection.md), zum Bearbeiten der Client-Verbindung, z. B. [Daten senden](../tcp-connection/send.md), [Verbindung schließen](../tcp-connection/close.md) usw.

## Beispiel
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "Puffer voll und nicht erneut senden\n";
};
// Worker starten
Worker::runAll();
```

Hinweis: Neben dem Verwenden einer anonymen Funktion als Rückruf können auch [andere Rückrufmethoden hier](../faq/callback_methods.md) verwendet werden.

## Siehe auch
onBufferDrain: Wird ausgelöst, wenn alle Daten im Anwendungsschicht-Sende-Puffer der Verbindung gesendet wurden
