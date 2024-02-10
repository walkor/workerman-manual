كيفية إنشاء خدمة UDP
إنشاء خدمة UDP في Workerman بسيط للغاية، مشابه للكود التالي

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

ملاحظة: بسبب الطبيعة غير المتصلة لبروتوكول UDP، فإن خدمة UDP لا تحتوي على أحداث onConnect و onClose.
