# خادم GlobalData المكون الخدمي

**``` (يتطلب إصدار Workerman> = 3.3.0) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

إنشاء كائن خدمة \GlobalData\Server

### المعاملات
 ``` listen_ip ```

عنوان IP المحلي الذي يتم الاستماع إليه ، الافتراضي هو ```0.0.0.0```

 ``` listen_port ```

منفذ الاستماع ، الافتراضي هو 2207


## مثال
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// الاستماع إلى المنفذ
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
