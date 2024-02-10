# Сервер компонента GlobalData

**``` (требуется версия Workerman>=3.3.0) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

Создает экземпляр службы \GlobalData\Server.

### Параметры
 ``` listen_ip ```

IP-адрес локального узла для прослушивания. По умолчанию - ```0.0.0.0```

 ``` listen_port ```

Порт для прослушивания. По умолчанию - 2207

## Пример
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// прослушиваемый порт
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
