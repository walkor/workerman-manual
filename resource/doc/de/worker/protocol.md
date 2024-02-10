# Protokoll

Erfordert ```(workerman >= 3.2.7)```

## Beschreibung:
```php
string Worker::$protocol
```

Legt die Protokollklasse für die aktuelle Worker-Instanz fest.

Hinweis: Das Protokollbearbeitungsklasse kann direkt beim Initialisieren des Worker beim Überwachen angegeben werden. Zum Beispiel
```php
$worker = new Worker('http://0.0.0.0:8686');
```



## Beispiel


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Worker ausführen
Worker::runAll();
```

Der obige Code entspricht dem folgenden Code


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * Zuerst wird überprüft, ob der Benutzer eine benutzerdefinierte \Protocols\Http-Protokollklasse hat.
 * Wenn nicht, wird die in Workerman eingebaute Protokollklasse Workerman\Protocols\Http verwendet.
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Worker ausführen
Worker::runAll();
```
