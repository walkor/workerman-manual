# 채널 컴포넌트 서버

**``` (Workerman 버전>=3.3.0 필요) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

\Channel\Server 서버를 인스턴스화합니다.

### 매개변수
``` listen_ip ```

리스닝할 로컬 IP 주소입니다. 전달되지 않으면 기본값은 ```0.0.0.0```입니다.

``` listen_port ```

리스닝할 포트입니다. 전달되지 않으면 기본값은 2206입니다.

## 예제

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 매개변수를 전달하지 않으면 기본적으로 0.0.0.0:2206을 리스닝합니다.
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
