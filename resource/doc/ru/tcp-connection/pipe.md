```php
# pipe
## Описание:
```php
void Connection::pipe(TcpConnection $target_connection)
```

## Параметры
Перенаправляет поток данных текущего соединения в целевое соединение. Встроенный контроль трафика. Этот метод очень полезен для создания TCP прокси.

## Пример TCP прокси

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// После установки TCP-соединения
$worker->onConnect = function(TcpConnection $connection)
{
    // Устанавливаем асинхронное соединение с локальным портом 80
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // Настройка направления данных текущего клиентского соединения к соединению на порт 80
    $connection->pipe($connection_to_80);
    // Настройка направления данных, возвращаемых соединением на порту 80, к клиентскому соединению
    $connection_to_80->pipe($connection);
    // Устанавливаем асинхронное соединение
    $connection_to_80->connect();
};

// Запуск worker
Worker::runAll();
```
