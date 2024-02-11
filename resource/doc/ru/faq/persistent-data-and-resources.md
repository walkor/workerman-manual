# Объекты и сохранение ресурсов
В традиционной веб-разработке объекты, данные, ресурсы, созданные на PHP, освобождаются после завершения запроса, что затрудняет достижение их постоянного хранения. Однако в Workerman это можно легко реализовать.

В Workerman можно легко осуществить постоянное хранение данных и ресурсов в памяти, поместив их в глобальные переменные или статические члены класса.

Приведенный ниже пример кода демонстрирует это:

Используется глобальная переменная ```$connection_count```, чтобы хранить текущее количество клиентских соединений в процессе.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Глобальная переменная для хранения текущего количества клиентских соединений в процессе
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // При подключении нового клиента увеличиваем количество соединений на 1
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // При закрытии клиента уменьшаем количество соединений на 1
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};

```

## См. Область видимости переменных в PHP:
https://php.net/manual/zh/language.variables.scope.php
