```php
# Workerman işlendikten sonra belirli bir isteği yeniden başlatmak için nasıl ayarlanır
Workerman'ı daha da basitleştirmek için doğrudan bu ayarı sağlamaz, ancak bu işlevi birkaç satır kodla gerçekleştirebilirsiniz.
```php
$worker->onMessage = function($connection, $data) {
    static $request_count;
    // İşlem işlemleri
    if(++$request_count > 10000) {
        // 10000 istek sayısına ulaşıldığında mevcut işlemi durdurun, ana işlem otomatik olarak yeni bir işlem başlatacaktır.
        Worker::stopAll();
    }
};
```
