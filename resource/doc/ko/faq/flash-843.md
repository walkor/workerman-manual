# Flash를 위한 843 포트 활성화

Flash가 원격 서버와의 소켓 연결을 시도할 때, 먼저 해당 서버의 843 포트로 보안 정책 파일을 요청합니다. 그렇지 않으면 Flash는 서버와의 연결을 설정할 수 없습니다. Workerman에서는 다음과 같은 방법으로 843 포트를 열고 보안 정책 파일을 반환할 수 있습니다.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$flash_policy = new Worker('tcp://0.0.0.0:843');
$flash_policy->onMessage = function(TcpConnection $connection, $message)
{
    $connection->send('<?xml version="1.0"?><cross-domain-policy><site-control permitted-cross-domain-policies="all"/><allow-access-from domain="*" to-ports="*"/></cross-domain-policy>'."\0");
};

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```

여기서 xml의 안전 정책 내용은 사용자의 요구에 따라 사용자 정의 설정할 수 있습니다.
