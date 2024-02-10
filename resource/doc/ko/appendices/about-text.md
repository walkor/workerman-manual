# 텍스트 프로토콜
> Workerman은 텍스트 프로토콜이라고 불리는 특정한 텍스트 프로토콜을 정의했습니다. 이 프로토콜 형식은 ```데이터 패킷 + 줄 바꿈```으로, 각 데이터 패킷의 끝에 줄 바꿈 문자를 추가하여 패킷의 종료를 나타냅니다.

예를 들어 아래의 buffer1 및 buffer2 문자열은 text 프로토콜을 준수합니다:

```php
// 텍스트에 줄 바꿈 추가
$buffer1 = 'abcdefghijklmn
';
// PHP에서 이중 인용부호 내의 \n은 줄 바꿈 문자를 나타내며, 예를 들어 "\n"과 같이 사용됩니다.
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// 서버와 소켓 연결 설정
$client = stream_socket_client('tcp://127.0.0.1:5678');
// buffer1 데이터를 text 프로토콜로 전송
fwrite($client, $buffer1);
// buffer2 데이터를 text 프로토콜로 전송
fwrite($client, $buffer2);
```

텍스트 프로토콜은 매우 간단하고 사용하기 쉽습니다. 개발자가 자체 프로토콜을 필요로 하는 경우, 예를 들어 모바일 앱과의 데이터 전송이나 하드웨어 통신 등, 텍스트 프로토콜을 사용하는 것이 편리하고 개발 및 디버깅이 매우 간편합니다.

**텍스트 프로토콜 디버깅**

> 텍스트 프로토콜을 사용하여 telnet 클라이언트 디버깅이 가능하며, 예를 들어 다음과 같은 예제가 있습니다:

새 파일 test.php 생성

```php
require_once __DIR__ . '/Workerman/Autoloader.php';
use Workerman\Worker;

$text_worker = new Worker("text://0.0.0.0:5678");

$text_worker->onMessage =  function($connection, $data)
{
    var_dump($data);
    $connection->send("hello world");
};

Worker::runAll();
```

```php test.php start```를 실행한 결과는 다음과 같습니다.

```php
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

새로운 터미널을 열어 telnet 테스트(리눅스 시스템의 telnet을 권장함)

로컬 테스트를 가정하면,
터미널에서 telnet 127.0.0.1 5678 실행
그 후 hi를 입력하고 엔터를 누르면 hello world\n 데이터를 수신합니다.

```telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
