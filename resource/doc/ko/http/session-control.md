# 설명
workerman은 4.x 버전부터 HTTP 서비스를 강화했습니다. 요청 클래스, 응답 클래스, 세션 클래스 및 [SSE](SSE.md)을 도입했습니다. Workerman의 HTTP 서비스를 사용하려는 경우, workerman 4.x 이상의 최신 버전을 강력히 권장합니다.

**다음은 workerman 4.x 버전의 사용 방법으로, workerman 3.x와 호환되지 않습니다.**

## 세션 저장 엔진 변경
workerman은 세션을 위해 파일 저장 엔진과 레디스 저장 엔진을 제공합니다. 기본적으로 파일 저장 엔진을 사용합니다. 레디스 저장 엔진으로 변경하려는 경우 다음 코드를 참조하십시오.
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// 레디스 설정
$config = [
    'host'     => '127.0.0.1', // 필수 매개변수
    'port'     => 6379,        // 필수 매개변수
    'timeout'  => 2,           // 선택적 매개변수
    'auth'     => '******',    // 선택적 매개변수
    'database' => 1,           // 선택적 매개변수
    'prefix'   => 'session_'   // 선택적 매개변수
];
// Workerman\Protocols\Http\Session::handlerClass 메서드를 사용하여 세션의 하부 드라이브 클래스 변경
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```


## 세션 저장 위치 설정
기본 저장 엔진을 사용하는 경우 세션 데이터는 기본적으로 디스크에 저장되며, 기본 위치는 `session_save_path()`의 반환 위치입니다.
다음 방법을 사용하여 저장 위치를 변경할 수 있습니다.
```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// 세션 파일 저장 위치 설정
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// worker 실행
Worker::runAll();
```


## 세션 파일 정리
기본 세션 저장 엔진을 사용하는 경우 디스크에 여러 세션 파일이 생성됩니다. Workerman은 php.ini에서 설정한 `session.gc_probability`, `session.gc_divisor`, `session.gc_maxlifetime` 옵션에 따라 만료된 세션 파일을 정리합니다. 이 세 가지 옵션에 대한 자세한 설명은 [php 매뉴얼](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability)을 참조하십시오.

## 저장 드라이버 변경
파일 세션 저장 엔진과 레디스 세션 저장 엔진 외에도, workerman은 표준 [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) 인터페이스를 통해 mangoDb 세션 저장 엔진, MySQL 세션 저장 엔진 등을 추가할 수 있습니다.

**새로운 세션 저장 엔진 추가 과정**
  1. [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) 인터페이스 구현
  2. `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` 메서드를 사용하여 하부 SessionHandler 인터페이스를 대체

**SessionHandlerInterface 구현**

사용자 정의 세션 저장 드라이버는 SessionHandlerInterface 인터페이스를 구현해야 합니다. 이 인터페이스에는 다음 메서드가 포함됩니다.
```php
SessionHandlerInterface {
    /* Methods */
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**SessionHandlerInterface 설명**
 - read 메서드는 저장에서 session_id에 해당하는 모든 세션 데이터를 읽습니다. 데이터를 직접 역직렬화하지 마십시오. 프레임워크가 자동으로 처리합니다.
 - write 메서드는 session_id에 해당하는 세션 데이터를 저장합니다. 데이터를 직렬화하지 마십시오. 프레임워크가 이미 처리했습니다.
 - destroy 메서드는 session_id에 해당하는 세션 데이터를 삭제합니다.
 - gc 메서드는 만료된 세션 데이터를 삭제합니다. 저장은 maxlifetime보다 큰 수정 시간을 가진 모든 세션에 대해 삭제 작업을 수행해야 합니다.
 - close는 아무 작업을 하지 않고 true를 반환해야 합니다.
 - open은 아무 작업을 하지 않고 true를 반환해야 합니다.

**하부 드라이버 변경**

SessionHandlerInterface를 구현한 후, 다음 메서드를 사용하여 세션의 하부 드라이버를 변경할 수 있습니다.
```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name은 SessionHandlerInterface를 구현한 SessionHandler 클래스의 이름입니다. 네임스페이스가 있는 경우 완전한 네임스페이스를 사용해야 합니다.
 - $config는 SessionHandler 클래스의 생성자 매개변수입니다.

**구체적인 구현**

*참고: 이 MySessionHandler 클래스는 하부 드라이버를 변경하는 과정을 설명하기 위한 것으로, 실제로는 상용 환경에 사용할 수 없습니다.*
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

class MySessionHandler implements SessionHandlerInterface
{
    protected static $store = [];
    
    public function __construct($config) {
        // ['host' => 'localhost']
        var_dump($config);
    }
   
    public function open($save_path, $name)
    {
        return true;
    }

    public function read($session_id)
    {
        return isset(static::$store[$session_id]) ? static::$store[$session_id]['content'] : '';
    }

    public function write($session_id, $session_data)
    {
        static::$store[$session_id] = ['content' => $session_data, 'timestamp' => time()];
    }

    public function close()
    {
        return true;
    }

    public function destroy($session_id)
    {
        unset(static::$store[$session_id]);
        return true;
    }

    public function gc($maxlifetime) {
        $time_now = time();
        foreach (static::$store as $session_id => $info) {
            if ($time_now - $info['timestamp'] > $maxlifetime) {
                unset(static::$store[$session_id]);
            }
        }
    }
}

//  가정: 새롭게 구현된 SessionHandler 클래스에는 구성이 필요합니다.
$config = ['host' => 'localhost'];
//  Workerman\Protocols\Http\Session::handlerClass($class_name, $config) 메서드를 사용하여 세션의 하부 드라이버 클래스를 변경합니다.
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
