# 통신 프로토콜의 역할
TCP는 데이터를 스트림으로 처리하기 때문에 클라이언트가 서버로 전송한 요청 데이터는 물 흐르듯이 서버로 흘러들어가게 됩니다. 서버는 데이터가 도착했을 때 데이터가 완전한지 확인해야 합니다. 왜냐하면 서버로 도착한 것이 한 요청의 일부일 수도 있고 때로는 여러 요청이 연속해서 도착하는 경우도 있을 수 있기 때문입니다. 따라서 요청이 모두 도착했는지, 또는 여러 개의 요청이 섞여있는 경우에는 통신 프로토콜을 정의하여 판단해야 합니다.

## WorkerMan에서 왜 프로토콜을 정의해야 하는가?

전통적인 PHP 개발은 주로 웹 기반으로 이루어지며, 대부분이 HTTP 프로토콜을 기반으로 합니다. HTTP 프로토콜의 분석과 처리는 대부분 WebServer가 별도로 담당하기 때문에 개발자는 프로토콜과 관련된 부분에 대해 걱정하지 않아도 됩니다. 그러나 우리가 HTTP 이외의 프로토콜을 기반으로 개발해야 할 경우에는 개발자가 프로토콜에 대해 고려해야 합니다.

## WorkerMan에서 지원하는 프로토콜
WorkerMan은 현재 HTTP, websocket, text 프로토콜(부록 참조), frame 프로토콜(부록 참조), ws 프로토콜(부록 참조)을 지원하고 있습니다. 이러한 프로토콜을 기반으로 통신해야 할 경우에는 직접 사용할 수 있습니다. 사용 방법은 Worker를 초기화할 때 프로토콜을 명시하는 것입니다. 예를 들면,
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket://0.0.0.0:2345 는 2345 포트에서 websocket 프로토콜을 사용한다는 것을 나타냅니다.
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// text 프로토콜
$text_worker = new Worker('text://0.0.0.0:2346');

// frame 프로토콜
$frame_worker = new Worker('frame://0.0.0.0:2347');

// tcp Worker, 소켓 전송을 기반으로 하며 어떤 응용 계층 프로토콜도 사용하지 않습니다.
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// udp Worker, 어떤 응용 계층 프로토콜도 사용하지 않습니다.
$udp_worker = new Worker('udp://0.0.0.0:2349');

// unix domain Worker, 어떤 응용 계층 프로토콜도 사용하지 않습니다.
$unix_worker = new Worker('unix:///tmp/wm.sock');
```

## 사용자 정의 통신 프로토콜 사용
만약 WorkerMan의 기본 통신 프로토콜이 개발 요구를 충족하지 못할 때, 개발자는 자체 통신 프로토콜을 정의할 수 있습니다. 정의 방법은 다음 절에 설명되어 있습니다.

**팁:**
Workerman에는 text 프로토콜이 내장되어 있어 텍스트와 개행 문자로 구성됩니다. 텍스트 프로토콜을 사용하면 대부분의 사용자 정의 프로토콜 신환이 가능하며, telnet 디버깅을 지원합니다. 개발자가 자체 응용 프로토콜을 개발하고자 할 때는 text 프로토콜을 직접 사용할 수 있으며 별도로 개발할 필요가 없습니다.

text 프로토콜에 대한 설명은 "부록 Text 프로토콜 부분"을 참조하세요.
