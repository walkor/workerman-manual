# 생성자 __construct

## 설명:
```php
Worker::__construct([string $listen , array $context])
```

Worker 컨테이너 인스턴스를 초기화하고 컨테이너의 여러 속성 및 콜백 인터페이스를 설정하여 특정 기능을 완료합니다.

## 매개변수
#### **``` $listen ```** (선택 사항, 지정하지 않으면 어떠한 포트도 수신하지 않음)

`$listen` 매개변수를 설정하면 소켓 수신을 실행합니다.

`$listen` 형식: <프로토콜>://<수신 주소>

**<프로토콜>은 다음 형식일 수 있습니다:**

tcp: 예시 ```tcp://0.0.0.0:8686```

udp: 예시 ```udp://0.0.0.0:8686```

unix: 예시 ```unix:///tmp/my_file``` ```(Workerman>=3.2.7 버전이 필요)```

http: 예시 ```http://0.0.0.0:80```

websocket: 예시 ```websocket://0.0.0.0:8686```

text: 예시 ```text://0.0.0.0:8686``` ```(text는 Workerman의 내장된 텍스트 프로토콜이며 telnet과 호환됨, 세부 내용은 부록 텍스트 프로토콜 부분 참조)```

그리고 다른 사용자 정의 프로토콜은 본 문서의 정의된 통신 프로토콜 부분을 참조

**<수신 주소>는 다음 형식일 수 있습니다:**

만약 unix 소켓일 경우 주소는 로컬 디스크 경로입니다.

unix 소켓이 아닐 경우, 주소 형식은 <로컬 아이피>:<포트 번호>

<로컬 아이피>는```0.0.0.0```으로 설정하여 모든 네트워크 카드(내부 아이피, 외부 아이피 및 로컬 루프백 127.0.0.1)를 수신함을 의미합니다.

<로컬 아이피>가```127.0.0.1```인 경우, 로컬 루프백만 수신하며 외부에서 접근할 수 없습니다.

<로컬 아이피>가 내부 아이피(예:```192.168.xx.xx```)인 경우, 내부 아이피만 수신하여 외부 사용자가 접근할 수 없습니다.

<로컬 아이피>가 로컬 아이피에 속하지 않은 경우 수신할 수 없으며 ```Cannot assign requested address``` 오류가 표시됩니다.

**참고:** <포트 번호>는 65535를 초과할 수 없습니다. <포트 번호>가 1024보다 작으면 수신하려면 관리자 권한이 필요하며, 수신할 포트는 이미 사용 중인 로컬하지 않은 포트여야 하며, 그렇지 않으면 ```Address already in use```오류가 표시됩니다.


#### **``` $context ```**

배열입니다. 소켓의 컨텍스트 옵션을 전달하는 데 사용합니다. [소켓 컨텍스트 옵션](https://php.net/manual/zh/context.socket.php) 참조

## 예제

Worker가 http 컨테이너로 http 요청을 처리하는 것을 수신하도록 하는 샘플 코드입니다.
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send("hello");
};

// worker 실행
Worker::runAll();
```

Worker가 websocket 컨테이너로 websocket 요청을 처리하는 것을 수신하도록 하는 샘플 코드입니다.
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// worker 실행
Worker::runAll();
```

Worker가 tcp 컨테이너로 tcp 요청을 처리하는 것을 수신하도록 하는 샘플 코드입니다.
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// worker 실행
Worker::runAll();
```

Worker가 udp 컨테이너로 udp 요청을 처리하는 것을 수신하도록 하는 샘플 코드입니다.
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// worker 실행
Worker::runAll();
```

Worker가 unix 도메인 소켓을 수신하도록 하는 샘플 코드입니다 ```(Workerman 버전>=3.2.7 요구)```.
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// worker 실행
Worker::runAll();
```

수신을 실행하지 않는 어떤 Worker 컨테이너는 일정한 작업을 처리하는 데 사용됩니다.
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 2.5초간격으로 실행
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// worker 실행
Worker::runAll();
```

**Worker가 사용자 정의 프로토콜을 수신하는 포트를 수신하는 예제**

최종 디렉토리 구조
```
├── Protocols              // 생성할 Protocols 디렉토리
│   └── MyTextProtocol.php // 생성할 사용자 정의 프로토콜 파일
├── test.php               // 생성할 테스트 스크립트
└── Workerman              // Workerman 소스 코드 디렉토리, 여기에 있는 코드를 수정하지 마십시오.
```

1. Protocols 디렉토리를 생성하고 프로토콜 파일을 생성합니다
   Protocols/MyTextProtocol.php (위의 디렉토리 구조를 참조)

```php
// 사용자 정의 프로토콜의 네임스페이스는 Protocols로 통일합니다.
namespace Protocols;
//간단한 텍스트 프로토콜, 프로토콜 포맷은 텍스트+개행
class MyTextProtocol
{
    // 패키지 분리 기능, 현재 패키지의 길이를 반환
    public static function input($recv_buffer)
    {
        // 개행 문자를 찾습니다.
        $pos = strpos($recv_buffer, "\n");
        // 개행 문자를 찾지 못하면 완전한 패키지가 아니므로 0을 반환하여 데이터를 계속 대기합니다.
        if($pos === false)
        {
            return 0;
        }
        // 개행 문자를 찾았다면 현재 패키지의 길이를 반환합니다. 개행 문자를 포함합니다.
        return $pos+1;
    }

    // 완전한 패키지를 받는 경우 자동으로 decode 메소드를 통해 디코딩하므로 여기서는 개행 문자를 자르기만 합니다.
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // 클라이언트에 데이터를 보내기 위해 encode 메소드를 통해 자동으로 인코딩되고 클라이언트에게 전송됩니다. 여기서는 개행 문자를 추가합니다.
    public static function encode($data)
    {
        return $data."\n";
    }
}
```
2. MyTextProtocol 프로토콜을 사용하여 요청을 수신하도록 test.php 파일을 생성합니다.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### MyTextProtocol worker ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * 완전한 데이터를 받은 후(마지막이 개행 문자) MyTextProtocol::decode('수신한 데이터')가 자동으로 실행되어
 * 결과가 $data로 전달됩니다.
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * 클라이언트에 데이터를 전송하면 자동으로 MyTextProtocol::encode('hello world') 메소드가 프로토콜을 인코딩하고,
     * 그 후 클라이언트에게 전송됩니다.
     */
    $connection->send("hello world");
};

// 모든 worker 실행
Worker::runAll();
```
3. 테스트

터미널을 열고 test.php가 있는 디렉토리로 이동한 후  `php test.php start`를 실행합니다.

```bash
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

터미널을 열고 telnet을 사용하여 테스트합니다. (preferably using telnet on a linux system)

로컬에서 테스트한다고 가정하고,
터미널에서 `telnet 127.0.0.1 5678`을 실행한 후 hi를 입력하고 엔터 키를 누르면 hello world\n 데이터를 수신할 것입니다.

```bash
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
