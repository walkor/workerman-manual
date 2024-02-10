# runAll
```php
void Worker::runAll(void)
```
Führt alle Worker-Instanzen aus.

**Hinweis:**

Nach Ausführung von Worker::runAll() erfolgt eine dauerhafte Blockierung, d.h. der Code nach Worker::runAll() wird nicht ausgeführt. Alle Worker-Instanzen sollten vor Worker::runAll() instanziiert werden.

### Parameter
Keine Parameter

### Rückgabewert
Kein Rückgabewert

## Beispiel: Ausführen mehrerer Worker-Instanzen

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// Führt alle Worker-Instanzen aus
Worker::runAll();
```


**Hinweis:**

Die Windows-Version von Workerman unterstützt nicht die Instanziierung mehrerer Worker im selben Datei. Das oben genannte Beispiel kann daher nicht unter der Windows-Version von Workerman ausgeführt werden.

Um die Windows-Version von Workerman zu verwenden, müssen mehrere Worker-Instanzen in separaten Dateien initialisiert werden, wie folgt:

start_http.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

// Führt alle Worker-Instanzen aus (hier gibt es nur eine Instanz)
Worker::runAll();
```


start_websocket.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// Führt alle Worker-Instanzen aus (hier gibt es nur eine Instanz)
Worker::runAll();
```
