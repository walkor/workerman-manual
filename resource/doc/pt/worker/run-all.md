# runAll
```php
void Worker::runAll(void)
```
Executa todas as instâncias do Worker.

**Nota:**

Depois que Worker::runAll() for executado, irá bloquear permanentemente, ou seja, o código abaixo de Worker::runAll() não será executado. Todas as instâncias do Worker devem ser inicializadas antes de Worker::runAll().

### Parâmetros
Sem parâmetros

### Retorno
Sem retorno

## Exemplo: Executar múltiplas instâncias do Worker

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

// Executar todas as instâncias do Worker
Worker::runAll();
```


**Nota:**

A versão do Workerman para Windows não suporta a instanciação de múltiplos Workers no mesmo arquivo. O exemplo acima não funcionará na versão do Workerman para Windows.

A versão do Workerman para Windows requer que a inicialização de várias instâncias do Worker seja feita em arquivos separados, como mostrado abaixo:

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

// Executar todas as instâncias do Worker (apenas uma instância aqui)
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

// Executar todas as instâncias do Worker (apenas uma instância aqui)
Worker::runAll();
```
