# Серверный компонент Channel

**``` (Требуется версия Workerman >=3.3.0) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

Создает экземпляр сервера \Channel\Server

### Параметры
 ``` listen_ip ```

IP-адрес локального прослушивания, если не указан, по умолчанию '0.0.0.0'

 ``` listen_port ```

Порт прослушивания, если не указан, по умолчанию 2206

## Пример

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Если параметры не переданы, по умолчанию слушается 0.0.0.0:2206
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
