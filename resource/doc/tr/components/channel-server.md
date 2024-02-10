# Kanal bileşeni sunucu tarafı

**``` (Workerman sürümü >=3.3.0 olmalıdır) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

Bir \Channel\Server sunucusu örneği oluşturur

### Parametreler
 ``` listen_ip ```

Yerel IP adresini dinler, belirtilmezse varsayılan ```0.0.0.0```'dır

 ``` listen_port ```

Dinlenen bağlantı noktası, belirtilmezse varsayılan 2206'dır

## Örnek

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Parametre belirtilmezse varsayılan olarak 0.0.0.0:2206'yı dinler
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
