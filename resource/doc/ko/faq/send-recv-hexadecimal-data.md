**16진수 데이터 송수신하기**

**16진수 데이터 수신**
데이터를 수신한 후 ```bin2hex($data)``` 함수를 사용하여 데이터를 16진수로 변환할 수 있습니다.

**16진수 데이터 전송**
데이터를 전송하기 전에 ```hex2bin($data)```를 사용하여 16진수 데이터를 이진수로 변환하고 전송할 수 있습니다.

**예시:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // 16진수 데이터 얻기
    $hex_data = bin2hex($data);
    // 클라이언트에게 16진수 데이터 전송
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```

