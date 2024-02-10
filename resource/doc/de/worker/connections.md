# Verbindungen
## Beschreibung:
```php
array Worker::$connections
```
Das Format ist
```php
array(id=>Verbindung, id=>Verbindung, ...)
```
Diese Eigenschaft enthält alle Client-Verbindungsobjekte **des aktuellen Prozesses**, wobei die ID die Kennung der Verbindung ist. Weitere Details finden Sie im Handbuch unter [Eigenschaft id von TcpConnection](../tcp-connection/id.md).

```$connections``` ist nützlich, um bei der Rundsendung oder beim Abrufen des Verbindungsobjekts anhand der Verbindungs-ID zu arbeiten.

Wenn Sie die Verbindungs-ID ```$id``` kennen, können Sie ganz einfach das entsprechende Verbindungsobjekt über ```$worker->connections[$id]``` abrufen und somit die entsprechende Socket-Verbindung bedienen, beispielsweise indem Sie ```$worker->connections[$id]->send('...')``` verwenden, um Daten zu senden.

Hinweis: Wenn eine Verbindung geschlossen wird (onClose ausgelöst), wird das entsprechende ```Verbindungs```-Objekt aus dem ```$connections```-Array entfernt.

Achtung: Entwickler sollten diese Eigenschaft nicht bearbeiten, da dies zu unvorhersehbaren Situationen führen kann.

## Beispiel
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// Beim Starten des Prozesses wird ein Timer gesetzt, um periodisch Daten an alle Client-Verbindungen zu senden
$worker->onWorkerStart = function($worker)
{
    // Timer, alle 10 Sekunden
    Timer::add(10, function()use($worker)
    {
        // Durchlaufen aller Client-Verbindungen des aktuellen Prozesses und Senden der aktuellen Zeit des Servers
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Worker ausführen
Worker::runAll();
```
