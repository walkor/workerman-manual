# transporte
## Descrição:
```php
string Worker::$transport
```

Define o protocolo de transporte utilizado pela instância atual do Worker. Atualmente, apenas 3 tipos são suportados (tcp, udp, ssl). Se não for definido, o padrão será tcp.

```Aviso: ssl requer Workerman versão>=3.3.7```

## Exemplo 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// Usando o protocolo udp
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// Executando o worker
Worker::runAll();
```

## Exemplo 2

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// O certificado deve ser preferencialmente um certificado válido
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // também pode ser um arquivo .crt
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// Aqui é configurado o protocolo websocket
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// Definindo transporte para ativar o ssl, websocket+ssl é wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
