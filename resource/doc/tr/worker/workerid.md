# id

Gereksinimler ```(workerman >= 3.2.1)``` 

## Açıklama:
```php
int Worker::$id
```

Mevcut worker işlemine ait ID numarası, ```0``` ile ```$worker->count-1``` arasında bir değere sahiptir.

Bu özellik, worker işlemlerini ayırt etmek için çok yararlıdır. Örneğin, bir worker örneğinde birden fazla işlem varsa ve geliştirici yalnızca bir işlemin içerisinde bir süreleyici ayarlamak istiyorsa, bu işlemi işlem numarası id'sini tanımlayarak yapabilir. Örneğin, yalnızca bu worker örneğindeki id numarası 0 olan işleçte bir süreleyici ayarlayabilir (örneğe bakınız).

## Not:

İşlem yeniden başlatıldığında, id numarası değeri değişmez.

İşlem numarası id, her bir worker örneğine dayalı bir dağıtımdır. Her worker örneği, kendi işlem numarasını 0'dan başlayarak atar, bu nedenle worker örnekleri arasında işlem numaraları çakışabilir, ancak bir worker örneğindeki işlem numaraları çakışmaz. Örneğin aşağıdaki örnek:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Worker örneği 1, 4 işleme sahiptir ve işlem id numaraları sırasıyla 0, 1, 2, 3 olacaktır.
$worker1 = new Worker('tcp://0.0.0.0:8585');
// 4 işlemi başlatmayı ayarla
$worker1->count = 4;
// Her işlem başladığında mevcut işlem id numarasını yani $worker1->id'yi yazdır
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// Worker örneği 2, 2 işleme sahiptir ve işlem id numaraları sırasıyla 0, 1 olacaktır.
$worker2 = new Worker('tcp://0.0.0.0:8686');
// 2 işlemi başlatmayı ayarla
$worker2->count = 2;
// Her işlem başladığında mevcut işlem id numarasını yani $worker2->id'yi yazdır
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// Worker'ı çalıştır
Worker::runAll();
```

Çıktı şuna benzer olacaktır:
```
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

Not: Windows sistemi, işlem sayısı olan count özelliğini desteklemediğinden sadece 0 numaralı bir id'ye sahiptir. Ayrıca aynı dosya üzerinde iki Worker dinleyici başlatma desteği olmadığı için bu örneği Windows sisteminde çalıştırmak mümkün değildir.


## Örnek
Bir worker örneğinde 4 işlem varsa, sadece id numarası 0 olan işleçte bir süreleyici ayarlayın.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // Sadece id numarası 0 olan işlemde bir süreleyici ayarla, diğer 1, 2, 3 numaralı işleçlerde süreleyici ayarlanmasın
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 worker işlemi, sadece 0 numaralı işlemde bir süreleyici ayarlandı\n";
        });
    }
};
// Worker'ı çalıştır
Worker::runAll();
```
