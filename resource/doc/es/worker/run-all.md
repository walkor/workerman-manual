# runAll
```php
void Worker::runAll(void)
```
Ejecuta todas las instancias de Worker.

**Nota:**

Worker::runAll() bloquea permanentemente la ejecución, lo que significa que el código después de Worker::runAll() no se ejecutará. Todas las instancias de Worker deben configurarse antes de Worker::runAll().

### Parámetros
Sin parámetros

### Valor de retorno
Sin valor de retorno

## Ejemplo: Ejecutar múltiples instancias de Worker

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

// Ejecutar todas las instancias de Worker
Worker::runAll();
```

**Nota:**

La versión de Windows de workerman no admite la instanciación de múltiples Workers en el mismo archivo.
El ejemplo anterior no funcionará en la versión de Windows de workerman.

La versión de Windows de workerman requiere que las múltiples instancias de Worker se inicialicen en archivos separados, como se muestra a continuación.

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

// Ejecutar todas las instancias de Worker (solo una instancia aquí)
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

// Ejecutar todas las instancias de Worker (solo una instancia aquí)
Worker::runAll();
```
