# GlobalData Komponenti Sunucu
** ``` (Workerman sürümü> = 3.3.0 olmalıdır) ``` **

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

\GlobalData\Server hizmetini örneklendirme

### Parametreler
 ``` listen_ip ```

Yerel IP adresini dinle, varsayılan olarak ```0.0.0.0```

 ``` listen_port ```

Portu dinle, varsayılan olarak 2207

## Örnek
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Port dinleme
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
