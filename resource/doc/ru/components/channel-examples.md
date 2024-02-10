```php
# Пример 1
**``` (требуется версия Workerman>=3.3.0) ```**

Много процессов (распределенный кластер) на основе Worker для системы рассылки, кластер отправки и кластерное вещание.

`start_channel.php`
Только одна служба start_channel может быть развернута во всей системе. Предположим, что она работает на 192.168.1.1.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Инициализация сервера каналов
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
Всю систему можно развернуть на нескольких службах start_ws, предположим, что они работают на двух серверах 192.168.1.2 и 192.168.1.3.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Сервер websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Подключение клиента к серверу каналов
    Channel\Client::connect('192.168.1.1', 2206);
    // Используем идентификатор своего процесса в качестве имени события
    $event_name = $worker->id;
    // Подписываемся на событие worker->id и регистрируем обработчик события
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "соединение не существует\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // Подписываемся на событие вещание
    $event_name = 'вещание';
    // При получении события вещание отправляем данные во все клиентские соединения внутри текущего процесса
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "workerID:{$worker->id} connectionID:{$connection->id} подключено\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
Всю систему можно развернуть на нескольких службах start_ws, предположим, что они работают на двух серверах 192.168.1.4 и 192.168.1.5.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Обработка http-запросов и отправка данных любому клиенту, требуется передача workerID и connectionID
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // Совместимость с workerman 4.x
    if (!is_array($request)) {
        $_GET = $request->get();
    }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // Отправка данных определенному рабочему процессу через определенное соединение
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // Глобальная рассылка данных
    else
    {
        $event_name = 'вещание';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```

## Тестирование 
1. Запустите службы на серверах.

2. Подключение клиентов к серверу

Откройте браузер Chrome, нажмите F12, откройте консоль, введите следующее (или поместите следующий код на html-страницу и запустите его с помощью js)

```javascript
// Можно также подключиться по адресу ws://192.168.1.3:4236
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("Получено сообщение сервера: " + e.data);
};
```

3. Отправка через http-интерфейс

Посетите URL-адрес ```http://192.168.1.4:4237/?content={$content}```  или  ```http://192.168.1.5:4237/?content={$content}``` для отправки данных ```$content``` всем соединениям клиентов

Посетите URL-адрес ```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` или ```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` для отправки данных ```$content``` определенному соединению в определенном рабочем процессе

Примечание: при тестировании замените ```{$worker_id}`, ```{$connection_id}``` и ```{$content}``` на фактические значения
```
