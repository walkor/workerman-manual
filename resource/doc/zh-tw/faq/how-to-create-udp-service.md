# 如何建立udp服務

在webman中建立udp服務非常簡單，類似如下程式碼：

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

注意：因為udp是無連線的，所以udp服務沒有onConnect和onClose事件。
