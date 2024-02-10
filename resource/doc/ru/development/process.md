# Основной процесс
（Пример с простым сервером чата WebSocket）

#### 1. Создание каталога проекта в любом месте 
Например, SimpleChat/
Перейти в каталог и выполнить `composer require workerman/workerman`

#### 2. Подключение `vendor/autoload.php` (создается после установки с помощью композера)
Создать start.php и подключить `vendor/autoload.php`
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. Выбор протокола
Здесь мы выбираем текстовый протокол (собственный протокол WorkerMan с форматом текст + переход на новую строку)

(В настоящее время WorkerMan поддерживает протоколы HTTP, Websocket, текстовый протокол. Если нужно использовать другие протоколы, обратитесь к главе о протоколах для разработки собственного протокола)

#### 4. Напишите входной запускающий скрипт по необходимости
Приведенный ниже пример - это простой файл входа для чата.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// При подключении клиента назначается UID, сохраняется соединение и уведомляются все клиенты
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // Назначаем UID для соединения
    $connection->uid = ++$global_uid;
}

// При получении сообщения от клиента отправляется всем
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// При отключении клиента отправляется всем клиентам
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}

// Создаем Worker для прослушивания интерфейса 2347 с текстовым протоколом
$text_worker = new Worker("text://0.0.0.0:2347");

// Запускать только 1 процесс, чтобы облегчить передачу данных между клиентами
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();
```

#### 5. Тестирование
С текстовым протоколом можно провести тестирование с помощью команды telnet
```shell
telnet 127.0.0.1 2347
```
