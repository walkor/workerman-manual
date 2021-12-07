# 如何建立udp服务

在workerman中建立udp服务很简单，类似如下代码

```php
$udp_worker = new Worker('udp://127.0.0.1:9090');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

注意：因为udp是无连接的，所以udp服务没有onConnect和onClose事件。
