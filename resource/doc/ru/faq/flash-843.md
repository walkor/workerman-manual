# Включение порта 843 для Flash

Когда Flash устанавливает сокетное подключение к удаленному серверу, он сначала запрашивает файл политики безопасности на соответствующем порту 843 на сервере. В противном случае Flash не сможет установить соединение с сервером. В Workerman можно использовать следующий метод для открытия порта 843 и возврата файла политики безопасности.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$flash_policy = new Worker('tcp://0.0.0.0:843');
$flash_policy->onMessage = function(TcpConnection $connection, $message)
{
    $connection->send('<?xml version="1.0"?><cross-domain-policy><site-control permitted-cross-domain-policies="all"/><allow-access-from domain="*" to-ports="*"/></cross-domain-policy>'."\0");
};

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```

Содержимое политики безопасности в формате XML можно настроить по вашему усмотрению.
