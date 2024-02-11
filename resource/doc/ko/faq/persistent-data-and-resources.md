# 객체 및 리소스의 지속성
전통적인 웹 개발에서 PHP로 생성된 객체, 데이터, 리소스 등은 요청이 완료되면 모두 해제되어 지속성을 유지하는 것이 어려웠습니다. 그러나 Workerman에서는 이러한 것들을 쉽게 지속시킬 수 있습니다.

Workerman에서는 메모리에 특정 데이터 리소스를 영구적으로 저장하려면 해당 리소스를 전역 변수나 클래스의 정적 멤버에 넣을 수 있습니다.

다음은 예시 코드입니다.

현재 프로세스의 클라이언트 연결 수를 저장하는 전역 변수 ```$connection_count```을 사용합니다.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 현재 프로세스의 클라이언트 연결 수를 저장하는 전역 변수
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // 새로운 클라이언트가 연결될 때마다 연결 수를 1 증가
    global $connection_count;
    ++$connection_count;
    echo "현재 연결 수: $connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // 클라이언트가 종료될 때마다 연결 수를 1 감소
    global $connection_count;
    $connection_count--;
    echo "현재 연결 수: $connection_count\n";
};
```

### PHP 변수 범위 참조:
https://www.php.net/manual/zh/language.variables.scope.php
