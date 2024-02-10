# UDP 서비스 구축 방법

Workerman에서 UDP 서비스를 구축하는 것은 매우 간단합니다. 다음과 같은 코드와 유사합니다.

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

참고: UDP는 연결이 없는 프로토콜이기 때문에 UDP 서비스에는 onConnect 및 onClose 이벤트가 없습니다.
