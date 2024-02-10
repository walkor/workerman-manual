# GlobalData 구성 요소 서버
**```(Workerman 버전>=3.3.0 이상 필요)```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

\GlobalData\Server 서비스를 인스턴스화합니다.

### 매개변수
``` listen_ip ```

리스닝할 로컬 IP 주소입니다. 전달되지 않으면 기본값은 ```0.0.0.0```입니다.

``` listen_port ```

리스닝할 포트입니다. 전달되지 않으면 기본값은 2207입니다.


## 예시
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 포트 수신
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
