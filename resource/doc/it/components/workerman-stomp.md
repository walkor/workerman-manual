# workerman/stomp

STOMP è un protocollo di comunicazione. Supporta la maggior parte delle code di messaggi come RabbitMQ, Apollo, ecc.

## Indirizzo del progetto:
https://github.com/walkor/stomp

## Installazione:
```php
composer require workerman/stomp
```

## Esempio
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
       // Iscrizione
        $client->subscribe('/topic/foo', function(Client $client, $data) {
            var_export($data);
        });
    };
    $client->onError = function ($e) {
        echo $e;
    };
    Timer::add(1, function () use ($client) {
        // Pubblicazione
        $client->send('/topic/foo', 'Ciao Workerman STOMP');
    });
    $client->connect();
};
Worker::runAll();
```
