## 프로토콜 사용자 정의 방법

사실 나만의 프로토콜을 정의하는 것은 매우 간단한 일입니다. 간단한 프로토콜은 보통 두 부분으로 구성됩니다.
 * 데이터 경계를 구분하는 식별자
 * 데이터 형식 정의

## 예시

### 프로토콜 정의
여기에서 데이터 경계를 구분하는 식별자로 줄 바꿈 문자 "\n"을 가정하고(json 데이터 내부에는 줄 바꿈 문자를 포함할 수 없음) 데이터 형식을 Json으로 가정하겠습니다. 아래는 이 규칙에 부합하는 요청 패킷의 예시입니다.

<pre>
{"type":"message","content":"hello"}
</pre>

위의 요청 데이터 끝에는 줄 바꿈 문자(이것은 PHP에서 **이중 인용부호** 문자열 "\n"로 표시됨)가 포함되어 있으며, 이는 요청의 종료를 의미합니다.

### 구현 단계
WorkerMan에서 위의 프로토콜을 구현하려면, 프로토콜을 JsonNL이라고 가정하고, 프로젝트를 MyApp으로 가정하면 다음 단계가 필요합니다.

1. 프로젝트의 Protocols 폴더에 프로토콜 파일을 넣어야 합니다. 예를들어, 파일은 MyApp/Protocols/JsonNL.php에 위치해야 합니다.

2. JsonNL 클래스를 구현해야 합니다. 네임 스페이스는 ```namespace Protocols;```이어야 하며, 반드시 input, encode, decode 세 가지의 정적 메서드를 구현해야 합니다.

주의: Workerman은 패킷 분리, 해석 및 패킹을 구현하기 위해 이러한 세 가지 정적 메서드를 자동으로 호출합니다. 구체적인 절차에 대한 내용은 아래의 실행 절차 설명을 참조하십시오.

### Workerman과 프로토콜 클래스 상호작용 흐름
1. 클라이언트가 서버에 데이터 패킷을 전송한다고 가정했을 때, 서버는 데이터(일부분일 수 있음)를 받은 즉시 프로토콜의 ```input```메소드를 호출하여 이 패킷의 길이를 확인합니다. ```input``` 메소드는 길이값 ```$length```를 Workerman 프레임워크에 반환합니다.

2. Workerman 프레임워크는 이 ```$length```값을 얻은 후 현재 데이터 버퍼에 ```$length```길이의 데이터가 이미 수신되었는지 확인합니다. 수신되지 않았다면 데이터가 수신될 때까지 계속해서 대기하게 됩니다.

3. 데이터 버퍼의 길이가 충분해지면, Workerman은 데이터 버퍼에서 ```$length```길이의 데이터를 잘라내어(즉 **패킷 분리**) 프로토콜의 ```decode```메소드를 호출하여 **패킷 해석**을 수행하고, 해석된 데이터는 ```$data```가 됩니다.

4. 해석 후, Workerman은 데이터 ```$data```를 콜백함수 ```onMessage($connection, $data)```에 전달하여 비즈니스에 전달합니다. 여기서 비즈니스는 ```$data``` 변수를 사용하여 클라이언트로부터 전송된 완전히 분할된 데이터를 받을 수 있습니다.

5. ```onMessage```에서 클라이언트에게 데이터를 전송해야 하는 경우, ```$connection->send($buffer)``` 메소드를 호출할 때, Workerman은 자동으로 프로토콜의 ```encode```메소드를 사용하여 ```$buffer```를 **패킹**하고 클라이언트에 전송합니다.

### 구체적인 구현

**MyApp/Protocols/JsonNL.php 구현 예시**

```php
namespace Protocols;
class JsonNL
{
    /**
     * 패킷의 완전성을 검사합니다.
     * 패킷 길이를 알 수 있는 경우 패킷의 전체 길이를 반환합니다.
     * 그렇지 않으면 0을 반환하여 더 많은 데이터가 필요함을 나타냅니다.
     * 문제가 있는 경우 false를 반환하면 현재 클라이언트 연결이 끊깁니다.
     * @param string $buffer
     * @return int
     */
    public static function input($buffer)
    {
        // 줄 바꿈 문자 "\n" 위치를 획득합니다.
        $pos = strpos($buffer, "\n");
        // 줄 바꿈 문자가 없으면 패킷 길이를 알 수 없으므로 0을 반환하여 더 많은 데이터가 필요함을 나타냅니다.
        if($pos === false)
        {
            return 0;
        }
        // 줄 바꿈 문자가 있으면 현재 패킷의 길이(줄 바꿈 문자 포함)를 반환합니다.
        return $pos+1;
    }

    /**
     * 패킹은 클라이언트에 데이터를 보낼 때 자동으로 호출됩니다.
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // Json 직렬화하고, 요청 종료 표시로 줄 바꿈 문자를 추가합니다
        return json_encode($buffer)."\n";
    }

    /**
     * 언패킹은 수신된 데이터 바이트 수(input 반환값보다 큰 양의 값)에 대해 자동으로 호출되며,
     * onMessage 콜백 함수의 $data 매개 변수로 전달됩니다.
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // 줄 바꿈 문자를 제거하고 배열로 복원합니다.
        return json_decode(trim($buffer), true);
    }
}
```

이로써, JsonNL 프로토콜의 구현이 완료되었으며, MyApp 프로젝트에서 사용할 수 있으며, 다음과 같은 방식으로 사용될 수 있습니다.

파일: MyApp\start.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data는 클라이언트에서 전송된 데이터이며 이미 JsonNL::decode로 처리되었습니다.
    echo $data;
    
    // $connection->send로 보내는 데이터는 자동으로 JsonNL::encode 방법을 사용하여 패킹된 후 클라이언트에게 전송됩니다.
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
...
```

> **팁**
> Workerman은 `Protocols` 네임스페이스 아래의 프로토콜을 로딩하려고 시도합니다. 예를들어, `new Worker('JsonNL://0.0.0.0:1234')`은 `Protocols\JsonNL` 프로토콜을 시도하게 됩니다.
> 만약 `Class 'Protocols\JsonNL' not found` 에러가 발생한다면, [자동 로딩](../faq/autoload.md)을 참조하여 자동으로 로딩할 수 있도록 구현해야 합니다.

### 프로토콜 인터페이스 설명
WorkerMan에서 개발하는 프로토콜 클래스는 반드시 input, encode, decode 세 가지의 정적 메서드를 구현해야 합니다. 프로토콜 인터페이스에 대한 자세한 설명은 Workerman/Protocols/ProtocolInterface.php를 참조하십시오. 다음과 같이 정의됩니다:

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

/**
 * Protocol interface
* @author walkor <walkor@workerman.net>
 */
interface ProtocolInterface
{
    /**
     * 수신 버퍼에서 패킷을 분리하는 데 사용됩니다.
     *
     * $recv_buffer에서 요청 패킷의 길이를 알 수 있는 경우 패킷의 전체 길이를 반환합니다.
     * 그렇지 않으면 0을 반환하여 현재 요청 패킷의 길이를 알 수 있는 더 많은 데이터가 필요함을 나타냅니다.
     * false나 음수를 반환하면 잘못된 요청으로 현재 클라이언트 연결이 끊겨집니다.
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * 요청 언패킹에 사용됩니다.
     *
     * input 반환값이 0보다 크고 WorkerMan이 충분한 데이터를 수신했다면 자동으로 decode가 호출되고, 이후 onMessage 콜백이 발생하며 decode로 해석된 데이터가 onMessage의 두 번째 매개 변수로 전달됩니다.
     * 즉, 완전한 클라이언트 요청을 받으면 자동으로 decode가 호출되고, 비즈니스 코드에서 수동으로 호출할 필요가 없습니다.
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     * 요청 패킹에 사용됩니다.
     *
     * 클라이언트에 데이터를 보내야하는 경우 $connection->send($data);를 호출하면 자동으로 $data를 encode로 한 번 패킹하여 프로토콜의 데이터 형식에 맞게 보내게 됩니다.
     * 즉, 클라이언트에게 보내는 데이터가 자동으로 encode로 패킹되며, 비즈니스 코드에서 수동으로 호출할 필요가 없습니다.
     * @param ConnectionInterface $connection
     * @param mixed $data
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## 주의사항:
Workerman은 프로토콜 클래스가 반드시 ProtocolInterface를 기반으로 구현해야 한다는 엄격한 규정은 없습니다. 실제로, 프로토콜 클래스에는 input, encode, decode 세 가지 정적 메서드만 포함되어 있으면 됩니다.
