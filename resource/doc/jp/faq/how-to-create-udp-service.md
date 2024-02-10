UDPサービスを設定する方法

WorkermanでUDPサービスを簡単に設定することができます。以下のコード例のようになります。

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

注意：UDPは接続を持たないため、UDPサービスにはonConnectやonCloseイベントはありません。
