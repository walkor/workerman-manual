# abonelikten çıkma
**``` (Workerman sürümü>=3.3.0 gereklidir) ```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```
Belirli bir olayın aboneliğini iptal eder. Bu olay gerçekleştiğinde, ```on($event_name, $callback)``` ile kayıt edilen ```$callback``` geri araması tetiklenmeyecektir.

### Parametre
``` $event_name ```

Olayın adı

### Dönüş Değeri
void

### Örnek
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
    $event_name = 'user_login';
    Channel\Client::on($event_name, function($event_data){
        var_dump($event_data);
    });
    Channel\Client::unsubscribe($event_name);
};

Worker::runAll();
```
