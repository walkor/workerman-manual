# PHP 콜백의 여러 가지 작성 방법
PHP에서 익명 함수를 사용하여 콜백을 작성하는 것이 가장 편리하지만, 익명 함수 방식 이외에도 PHP에서는 다른 콜백 작성 방법이 있습니다. 다음은 PHP에서의 여러 가지 콜백 작성 방법에 대한 예시입니다.

## 1. 익명 함수 콜백
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// 익명 함수 콜백
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // 브라우저로 'hello world'를 전송
    $connection->send('hello world');
};

Worker::runAll();
```

## 2. 보통 함수 콜백
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// 익명 함수 콜백
$http_worker->onMessage = 'on_message';

// 보통 함수
function on_message(TcpConnection $connection, Request $request)
{
    // 브라우저로 'hello world'를 전송
    $connection->send('hello world');
}

Worker::runAll();
```

## 3. 클래스 메소드로 콜백 사용
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public function __construct(){}
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
시작 스크립트 start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// MyClass 불러오기
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// 객체 생성
$my_object = new MyClass();

// 클래스 메소드 호출
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

참고:
위의 코드 구조에서는 생성자에서 MySQL 연결, Redis 연결, Memcache 연결 등과 같은 리소스를 초기화하는 것을 허용하지 않습니다. 왜냐하면 ```$my_object = new MyClass();```는 메인 프로세스에서 실행되기 때문입니다. MySQL을 예로 들면, 메인 프로세스에서 초기화된 MySQL 연결 및 기타 리소스는 자식 프로세스에서 상속됩니다. 각 자식 프로세스는 이 데이터베이스 연결을 조작할 수 있지만, 서비스 측의 이러한 연결은 동일한 연결에 해당하며, 예기치 않은 오류가 발생할 수 있습니다. 이러한 오류에는 ```mysql gone away``` 오류가 포함됩니다.

위의 코드 구조는 클래스 생성자에서 리소스를 초기화해야하는 경우 다음과 같이 작성할 수 있습니다.
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // 가정: 데이터베이스 연결 클래스는 MyDbClass입니다.
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
시작 스크립트 start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// 클래스 초기화
$worker->onWorkerStart = function($worker) {
    // MyClass 불러오기
    require_once __DIR__.'/MyClass.php';
    
    // 객체 생성
    $my_object = new MyClass();

    // 클래스 메소드 호출
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

위의 코드 구조에서 onWorkerStart는 이미 자식 프로세스에 속하기 때문에 각 자식 프로세스는 각각의 MySQL 연결을 설정하므로 공유 연결 문제가 발생하지 않습니다. 이렇게 함으로써 또 다른 장점은 비즈니스 코드 리로드를 지원한다는 것입니다. MyClass.php가 자식 프로세스에 로드되기 때문에 비즈니스 로직을 수정한 후에는 직접 리로드하여 적용할 수 있습니다.

## 4. 클래스의 정적 메소드를 콜백으로 사용
정적 MyClass.php 클래스
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public static function onWorkerStart(Worker $worker){}
    public static function onConnect(TcpConnection $connection){}
    public static function onMessage(TcpConnection $connection, $message) {}
    public static function onClose(TcpConnection $connection){}
    public static function onWorkerStop(Worker $worker){}
}
```
시작 스크립트 start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// MyClass 불러오기
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// 클래스의 정적 메소드 호출
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// 클래스에 네임스페이스가 있는 경우 다음과 같은 방식으로 작성합니다.
// $worker->onWorkerStart = array('your\namesapce\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('your\namesapce\MyClass', 'onConnect');
// $worker->onMessage     = array('your\namesapce\MyClass', 'onMessage');
// $worker->onClose       = array('your\namesapce\MyClass', 'onClose');
// $worker->onWorkerStop  = array('your\namesapce\MyClass', 'onWorkerStop');

Worker::runAll();
```

참고: PHP의 작동 방식에 따라 new를 사용하지 않으면 생성자가 호출되지 않으며, 또한 정적 클래스의 메서드 내에서는 ```$this```를 사용할 수 없습니다.
