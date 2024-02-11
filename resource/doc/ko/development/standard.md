# 개발 규칙

## 애플리케이션 디렉토리

애플리케이션 디렉토리는 임의의 위치에 놓을 수 있습니다.

## 진입 파일

Workerman 애플리케이션은 Workerman에서도 nginx+PHP-FPM에서와 마찬가지로 진입 파일이 필요하며, 진입 파일의 이름에 특별한 요구사항이 없으며, 이 진입 파일은 PHP CLI 방식으로 실행됩니다.

진입 파일에는 리스닝 프로세스를 생성하는 코드가 포함되어 있으며, 아래는 Worker를 사용한 개발 기반의 코드 예시입니다.

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 2345번 포트를 리스닝하는 Worker 생성, http 프로토콜을 사용
$http_worker = new Worker("http://0.0.0.0:2345");

// 외부로부터 서비스 제공을 위해 4개의 프로세스를 시작
$http_worker->count = 4;

// 브라우저로부터 데이터를 받으면 브라우저에 'hello world'를 응답
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // 브라우저로 'hello world' 전송
    $connection->send('hello world');
};

Worker::runAll();

```

## Workerman의 코드 규칙

1. 클래스는 대문자 카멜 표기법을 사용하며, 클래스 파일 이름은 내부 클래스 이름과 동일해야 하며, 자동으로로딩 될 수 있도록 합니다. 예시: 
   ```php
   class UserInfo
   {
   ...
   ```

2. 네임스페이스를 사용하고, 네임스페이스 이름은 디렉토리 경로와 일치하며, 개발자의 프로젝트 루트 디렉토리를 기준으로합니다.

   예를 들어 프로젝트 MyApp/인 경우, 클래스 파일 MyApp/MyClass.php는 프로젝트 루트 디렉토리에 있으므로 네임스페이스를 생략합니다. MyApp/Protocols/MyProtocol.php는 MyProtocol.php가 MyApp 프로젝트의 Protocols 디렉토리에 있으므로 ```namespace Protocols;```를 추가해야 합니다. 아래와 같습니다:
   ```php
   namespace Protocols;
   class MyProtocol
   {
   ....
   ```

3. 일반 함수 및 변수 이름은 소문자와 밑줄을 사용합니다. 예시:
   ```php
   $connection_list = array();
   function get_connection_list()
   {
   ....
   ```

4. 클래스 멤버 및 메소드는 소문자로 시작하고 카멜 표기법을 사용합니다. 예시:
   ```php
   public $connectionList;
   public function getConnectionList();
   ```

5. 함수 및 클래스의 매개변수는 소문자와 밑줄을 사용합니다. 예시:
   ```php
   function get_connection_list($one_param, $two_param)
   {
   ....

   ```
