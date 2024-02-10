# Workerman/Stomp

STOMP é um protocolo de comunicação. Ele suporta a maioria das filas de mensagens, como RabbitMQ, Apollo, etc.

## Endereço do projeto:
https://github.com/walkor/stomp

## Instalação:
```php
composer require workerman/stomp
```

## Exemplo
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Stomp\Client;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $client = new Workerman\Stomp\Client('stomp://127.0.0.1:61613');
    $client->onConnect = function(Client $client) {
       // Inscrever
        $client->subscribe('/topic/foo', function(Client $client, $data) {
            var_export($data);
        });
    };
    $client->onError = function ($e) {
        echo $e;
    };
    Timer::add(1, function () use ($client) {
        // Enviar
        $client->send('/topic/foo', 'Olá Workerman STOMP');
    });
    $client->connect();
};
Worker::runAll();
```
