# مكون الخادم للقناة

**``` (يتطلب إصدار Workerman >= 3.3.0) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

تطبيق خادم \Channel\Server

### المعاملات
``` listen_ip ```

عنوان IP المستمع المحلي ، إذا لم يتم تمريره فسيكون الافتراضي``` 0.0.0.0 ```

``` listen_port ```

منفذ الاستماع ، إذا لم يتم تمريره فسيكون الافتراضي 2206

## مثال

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// بدون تمرير المعاملات فإن الاستماع الافتراضي 0.0.0.0:2206
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
