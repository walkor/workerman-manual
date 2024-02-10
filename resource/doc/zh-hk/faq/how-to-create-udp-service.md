# 如何建立UDP服務

在webman中建立UDP服務非常簡單，類似以下的程式碼

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

注意：因為UDP是無連接的，所以UDP服務沒有onConnect和onClose事件。
