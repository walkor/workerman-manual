# stopAll
```php
void Worker::stopAll(void)
```

Stoppt den aktuellen Prozess und beendet ihn.

> **Hinweis**
> `Worker::stopAll()` wird verwendet, um den aktuellen Prozess zu stoppen. Nachdem der aktuelle Prozess beendet wurde, startet der Hauptprozess sofort einen neuen Prozess. Wenn Sie den gesamten Workerman-Dienst stoppen möchten, verwenden Sie `posix_kill(posix_getppid(), SIGINT)`.

### Parameter
Keine Parameter

### Rückgabewert
Kein Rückgabewert

## Beispiel max_request

Im folgenden Beispiel wird der Unterprozess nach der Bearbeitung von 1000 Anfragen mit stopAll beendet, um anschließend einen neuen vollständigen Prozess zu starten. Ähnlich wie die max_request-Eigenschaft von php-fpm dient dies hauptsächlich zur Behebung von Speicherlecks, die durch Fehler im PHP-Geschäftscode verursacht werden.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Jeder Prozess kann maximal 1000 Anfragen abarbeiten
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Anzahl der bereits behandelten Anfragen
    static $request_count = 0;

    $connection->send('hello http');
    // Wenn die Anzahl der Anfragen 1000 erreicht
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * Beendet den aktuellen Prozess. Der Hauptprozess startet sofort einen neuen vollständigen Prozess, um den Neustart des Prozesses abzuschließen.
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
