# on
**``` (требуется версия Workerman >=3.3.0) ```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
Подписаться на событие ```$event_name``` и зарегистрировать обратный вызов ```$callback_function``` при возникновении события.

## Параметры обратного вызова

``` $event_name ```

Название подписанного события, может быть любой строкой.

``` $callback_function ```

Функция обратного вызова, вызываемая при возникновении события. Прототип функции: ```callback_function(mixed $event_data)```. ```$event_data``` - данные события, передаваемые при его публикации.

Примечание:

Если одно и то же событие зарегистрировано дважды, вторая функция заменит первую.

## Пример
Worker с несколькими процессами (многопользовательский сервер), один клиент отправляет сообщение, которое транслируется всем клиентам.

start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Инициализация сервера каналов
$channel_server = new Channel\Server('0.0.0.0', 2206);

// Веб-сервер WebSocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// При каждом запуске процесса worker
$worker->onWorkerStart = function($worker)
{
    // Подключение клиента к серверу каналов
    Channel\Client::connect('127.0.0.1', 2206);
    // Подписка на событие трансляции и регистрация обратного вызова события
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // Трансляция сообщения всем клиентам процесса worker
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // Использование данных, отправленных клиентом, как данных события
   $event_data = $data;
   // Публикация события трансляции для всех процессов worker
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**Тестирование**

Откройте браузер Chrome, нажмите F12, чтобы открыть консоль отладки. В разделе Console введите (или разместите следующий код на HTML-странице и выполните его в JS).

Подключение для приема сообщений
```javascript
// Замените 127.0.0.1 на реальный IP-адрес Workerman
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("Получено сообщение от сервера: " + e.data);
};
```

Трансляция сообщения
```
ws.send('Привет, мир');
```
