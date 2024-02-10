```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
Belli bir işlevi veya sınıf yöntemini zamanlanmış olarak çalıştırır.

Not: Zamanlayıcı mevcut işlemde çalışır; workerman'da yeni bir işlem veya diğer zamanlayıcıları çalıştırmak için yeni bir iş parçacığı oluşturulmaz.

### Parametreler
 ``` time_interval ```

Ne sıklıkla çalıştırılacağı, saniye cinsinden, ondalıklı sayıları destekler, yani milisaniye hassasiyetine kadar kesinlikle belirtilebilir. 

 ``` callback ```

Geribildirim işlevi ```Not: Geri çağırma işlevi bir sınıf yöntemi ise yöntemin public olması gerekmektedir```

 ``` args ```

Geri çağırma işlevinin parametreleri, bir dizi olmalıdır, dizi elemanları parametre değerlerini içermelidir.

 ``` persistent ```

Kalıcı olup olmadığı, yalnızca bir kez zamanlanmış bir görevi çalıştırmak istiyorsanız false (`Timer::del()` çağrılmasına gerek yoktur). Varsayılan olarak true, yani sürekli olarak zamanlanmış görevi çalıştırır.

### Dönen Değer
Bir zamanlayıcıya ait `timerid` yi temsil eden bir tamsayı döndürür, `Timer::del($timerid)` çağrılarak bu zamanlayıcıyı yok edebilirsiniz.

### Örnekler

#### 1, Zamanlanmış işlev anonim bir fonksiyon (kapanış) olarak tanımlanmış
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Kaç tane iş parçacığında zamanlanmış görev çalıştırılacağına dikkat edin, iş akışı özelliğinin birden çok iş parçacığında eş zamanlı sorunlara sahip olup olmadığını kontrol edin.
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Her 2.5 saniyede bir çalıştır
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "görev çalıştır\n";
    });
};

// İşçiyi çalıştır
Worker::runAll();
```
