# GlobalData 구성 요소 클라이언트
**``` (Workerman 버전 >= 3.3.0 이상이 필요) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

\GlobalData\Client 클라이언트 객체를 인스턴스화합니다. 클라이언트 객체에 속성을 할당하여 프로세스간에 데이터를 공유할 수 있습니다.

### 매개변수
GlobalData 서버 주소는 ```<ip주소>:<포트>``` 형식으로 지정하며, 예를 들어 ```127.0.0.1:2207``` 입니다.

GlobalData 서버 클러스터인 경우 주소 배열을 전달해야 하며, 예를 들어 ```array('10.0.0.10:2207', '10.0.0.0.11:2207')``` 입니다.

## 설명
할당, 읽기, isset, unset 작업을 지원합니다.
동시에 cas 원자적 작업도 지원합니다.


## 예제

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// GlobalData 서버
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// 워커가 시작될 때
$worker->onWorkerStart = function()
{
    // 전역 global data 클라이언트를 초기화합니다.
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// 서버가 메시지를 받을 때마다
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // $global->somedata의 값을 변경하면 다른 프로세스도 이 변수를 공유합니다.
    global $global;
    echo "now global->somedata=".var_export($global->somedata, true)."\n";
    echo "set \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### 전체 사용법 (php-fpm 환경에서도 사용 가능)
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));

```

## 주의사항:
GlobalData 구성 요소는 MySQL 연결, 소켓 연결과 같은 리소스 유형의 데이터를 공유할 수 없습니다.

Workerman 환경에서 GlobalData/Client를 사용하는 경우 onXXX 콜백에서 GlobalData/Client 객체를 인스턴스화해야 합니다. 예를 들어, onWorkerStart에서 인스턴스화해야 합니다.

공유 변수를 다음과 같이 조작할 수 없습니다.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
다음과 같이 할 수 있습니다.
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
