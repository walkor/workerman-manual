## Wie man ein Protokoll anpasst

Die Erstellung eines eigenen Protokolls ist eigentlich recht einfach. Ein einfaches Protokoll enthält im Allgemeinen zwei Teile:
* Ein Trennzeichen zur Unterscheidung der Daten
* Die Definition des Datenformats

## Ein Beispiel

### Protokolldefinition
Angenommen, das Trennzeichen zur Unterscheidung der Daten ist der Zeilenumbruch "\n" (es sei darauf hingewiesen, dass die Anfrage selbst keine Zeilenumbrüche enthalten sollte) und das Datenformat ist JSON. Hier ist ein Beispiel für ein Paket, das diese Regel erfüllt.

<pre>
{"type":"message","content":"hello"}
</pre>

Bitte beachten Sie das Zeilenumbruchszeichen am Ende des Anfragepakets (im PHP-Code wird dies als **Doppelzitatszeichen**-Zeichenfolge "\n" dargestellt), das das Ende einer Anfrage kennzeichnet.

### Implementierungsschritte
Wenn Sie das oben genannte Protokoll in Workerman implementieren möchten, und angenommen, dass Ihr Protokoll JsonNL heißt und sich im Projekt MyApp befindet, müssen Sie die folgenden Schritte ausführen:

1. Platzieren Sie die Protokolldatei im Ordner Protokolle des Projekts, z.B. die Datei MyApp/Protocols/JsonNL.php

2. Implementieren Sie die Klasse JsonNL, wobei ```namespace Protocols;``` für den Namensraum verwendet wird. Es müssen drei statische Methoden implementiert werden: input, encode und decode.

Hinweis: Workerman wird diese drei statischen Methoden automatisch aufrufen, um die Paketierung, Entpackung und Verschlüsselung zu implementieren. Details zum Ablauf finden Sie in der nachstehenden Ablaufbeschreibung.

### Interaktionsablauf zwischen Workerman und der Protokollklasse
1. Angenommen, der Client sendet ein Datenpaket an den Server. Sobald der Server die Daten (möglicherweise teilweise) empfängt, ruft er sofort die Methode ```input``` des Protokolls auf, um die Länge des Pakets zu überprüfen. Die Methode ```input``` gibt die Länge ```$length``` an das Workerman-Framework zurück.
2. Nachdem das Workerman-Framework den Wert ```$length``` erhalten hat, überprüft es, ob im aktuellen Datenpuffer bereits eine Datenlänge von ```$length``` empfangen wurde. Falls nicht, wird weiter auf Daten gewartet, bis die Datenpufferlänge nicht kleiner als ```$length``` ist.
3. Wenn die Datenpufferlänge ausreicht, schneidet Workerman die Daten mit einer Länge von ```$length``` aus dem Puffer aus (d.h. **Paketierung**) und ruft die Methode ```decode``` des Protokolls zum **Entpacken** auf. Die entschlüsselten Daten werden in ```$data``` zurückgegeben.
4. Nach dem Entpacken übergibt Workerman die Daten ```$data``` im Rahmen des ```onMessage($connection, $data)```-Callbacks an die Geschäftslogik. Innerhalb des ```onMessage```-Callbacks kann die Geschäftslogik die vollständigen und bereits entschlüsselten Daten des Clients mithilfe der Variablen ```$data``` erhalten.
5. Wenn die Geschäftslogik in ```onMessage``` Daten an den Client senden muss, ruft Workerman automatisch die Methode ```encode``` des Protokolls auf, um das ```$buffer``` zu **verpacken**, bevor es an den Client gesendet wird.

### Spezifische Implementierung

**Implementierung von MyApp/Protocols/JsonNL.php**

```php
namespace Protocols;
class JsonNL
{
    /**
     * Überprüft die Vollständigkeit des Pakets
     * Gibt die Länge des Pakets im Puffer zurück, sofern sie ermittelt werden kann, andernfalls 0, um auf weitere Daten zu warten
     * Bei einem Protokollfehler kann false zurückgegeben werden, was zur Trennung der aktuellen Client-Verbindung führt
     * @param string $buffer
     * @return int
     */
    public static function input($buffer)
    {
        // Position des Zeilenumbruchs "\n" ermitteln
        $pos = strpos($buffer, "\n");
        // Wenn kein Zeilenumbruch vorhanden ist, kann die Paketlänge nicht bestimmt werden, daher 0 zurückgeben und auf weitere Daten warten
        if($pos === false)
        {
            return 0;
        }
        // Wenn ein Zeilenumbruch vorhanden ist, die aktuelle Paketlänge (einschließlich des Zeilenumbruchs) zurückgeben
        return $pos+1;
    }

    /**
     * Verpackungsmethode, die automatisch aufgerufen wird, wenn Daten an den Client gesendet werden sollen
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // JSON-Serialisierung, gefolgt von einem Zeilenumbruch als Abschlusskennung für die Anfrage
        return json_encode($buffer)."\n";
    }

    /**
     * Entpackungsmethode, die automatisch aufgerufen wird, wenn die empfangenen Daten die Länge des vom input zurückgegebenen Werts erreicht haben (Wert größer als 0)
     * und gibt die vom onMessage-Callback erhaltenen Daten im $data-Parameter zurück
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // Zeilenumbruch entfernen und in ein Array umwandeln
        return json_decode(trim($buffer), true);
    }
}
```

Damit ist das JsonNL-Protokoll implementiert und kann in der MyApp-Anwendung verwendet werden. Ein Beispiel zur Verwendung finden Sie unten.

Datei: MyApp\start.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data enthält die vom Client übertragenen Daten, die bereits von JsonNL::decode verarbeitet wurden
    echo $data;
    
    // Die mit $connection->send gesendeten Daten werden automatisch mit der JsonNL::encode-Methode verpackt und an den Client gesendet
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
...
```

> **Hinweis**
> Workerman versucht, Protokolle im Namensraum `Protocols` zu laden. Beispielsweise versucht `new Worker('JsonNL://0.0.0.0:1234')` das Protokoll `Protocols\JsonNL` zu laden.
> Wenn der Fehler `Class 'Protocols\JsonNL' not found` auftritt, lesen Sie bitte die Anleitung zur [automatischen Ladung](../faq/autoload.md) der Klassen.

### Protokoll-Schnittstellenbeschreibung
Bei der Entwicklung von Protokollklassen in Workerman müssen drei statische Methoden implementiert werden: input, encode und decode. Die Schnittstellenbeschreibung für Protokolle finden Sie in Workerman/Protocols/ProtocolInterface.php und lautet wie folgt:

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

/**
 * Protokollschnittstelle
* @author walkor <walkor@workerman.net>
 */
interface ProtocolInterface
{
    /**
     * Zum Aufteilen des recv_buffer in Pakete
     *
     * Wenn die Länge des Anfragepakets aus $recv_buffer ermittelt werden kann, wird die Gesamtlänge zurückgegeben
     * Andernfalls wird 0 zurückgegeben, was bedeutet, dass mehr Daten benötigt werden, um die Länge des aktuellen Anfragepakets zu bestimmen
     * Wenn false oder eine negative Zahl zurückgegeben wird, wird dies als ungültige Anfrage betrachtet und die Verbindung wird getrennt
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * Zum Entpacken von Anfragen
     *
     * Wenn input einen Wert größer als 0 zurückgibt und Workerman genügend Daten empfangen hat, wird decode automatisch aufgerufen
     * Anschließend wird das onMessage-Callback ausgelöst und die von decode entschlüsselten Daten als zweites Argument an das onMessage-Callback übergeben
     * Mit anderen Worten, wenn eine vollständige Client-Anfrage empfangen wird, wird decode automatisch zur Entschlüsselung aufgerufen und muss nicht manuell im Geschäftscode aufgerufen werden
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     * Zum Verpacken von Anfragen
     *
     * Wenn Daten an den Client gesendet werden sollen (z.B. mit $connection->send($data);), wird die encode-Methode automatisch verwendet, um die Daten in das entsprechende Protokollformat zu verpacken, bevor sie an den Client gesendet werden
     * Mit anderen Worten, die an den Client gesendeten Daten werden automatisch durch encode verpackt und müssen nicht manuell im Geschäftscode aufgerufen werden
     * @param ConnectionInterface $connection
     * @param mixed $data
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## Hinweis:
Workerman schreibt nicht vor, dass Protokollklassen die ProtocolInterface implementieren müssen. In der Praxis genügt es, wenn die Klasse die drei statischen Methoden input, encode und decode enthält.
