# runAll
```php
void Worker::runAll(void)
```
Esegue tutte le istanze dei Worker.

**Nota:**

Dopo l'esecuzione di Worker::runAll(), ci sarà un blocco permanente, il che significa che il codice successivo a Worker::runAll() non verrà eseguito. Tutte le istanze dei Worker dovrebbero essere istanziate prima di Worker::runAll().

### Parametri
Nessun parametro

### Valore restituito
Nessun valore restituito

## Esempio: Esecuzione di più istanze dei Worker

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

// Esegue tutte le istanze dei Worker
Worker::runAll();
```


**Nota:**

La versione Windows di workerman non supporta l'istanziazione di più Worker nello stesso file.
L'esempio sopra non può essere eseguito nella versione Windows di workerman.

La versione Windows di workerman richiede di inizializzare le diverse istanze dei Worker in file separati, come mostrato di seguito

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

// Esegue tutte le istanze dei Worker (qui c'è solo un'istanza)
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

// Esegue tutte le istanze dei Worker (qui c'è solo un'istanza)
Worker::runAll();
```
