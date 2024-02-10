## nginx/apache 프록시를 통해 클라이언트의 실제 IP 주소를 어떻게 가져오나요?

nginx/apache를 workerman 프록시로 사용하면, 실제로 nginx/apache가 workerman의 클라이언트 역할을 하므로 workerman에서 가져오는 클라이언트 IP는 실제 클라이언트 IP가 아닌 nginx/apache 서버의 IP가 됩니다. 클라이언트의 실제 IP를 가져오는 방법은 다음과 같습니다.

**동작 원리:**

nginx/apache는 클라이언트의 실제 IP를 http 헤더를 통해 전달합니다. 예를 들어 nginx 구성에서 location에 ```proxy_set_header X-Real-IP $remote_addr;```을 설정합니다. workerman은 이 헤더 값을 읽어서 이 값을 ```$connection 객체에 저장```합니다 (GatewayWorker의 경우 ```$_SESSION``` 변수에 저장), 필요할 때 변수를 직접 읽으면 됩니다.

**주의 사항:**

아래 구성은 http/https ws/wss 프로토콜에 적합합니다. 다른 프로토콜의 클라이언트 IP를 가져오는 방법도 유사하며, 프록시 서버가 실제 클라이언트 IP를 데이터 패킷에 삽입하여 전달해야 합니다.

**다음은 nginx 구성의 예시:**

```nginx
server {
  listen 443;

  ssl on;
  ssl_certificate /etc/ssl/server.pem;
  ssl_certificate_key /etc/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /wss
  {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    # 이 부분은 http 헤더를 통해 실제 클라이언트 IP를 전달함
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} 사이트의 다른 구성...
}
```

**workerman에서 nginx로 설정한 헤더를 읽어 클라이언트 IP를 가져오기**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// 클라이언트가 연결되면, TCP 3-way handshake가 완료된 후에 호출됩니다.
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * 클라이언트 websocket 핸드쉐이크 시 onWebSocketConnect 콜백을 통해 nginx가 http 헤더로 전달한 X_REAL_IP 값을 가져옵니다.
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * connection 객체에는 realIP 속성이 없습니다. 여기서 connection 객체에 동적으로 realIP 속성을 추가합니다.
        * 기억하세요, PHP 객체에는 속성을 동적으로 추가할 수 있으며, 원하는 속성 이름을 사용할 수도 있습니다.
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 클라이언트 실제 IP를 사용할 때, $connection->realIP를 직접 사용합니다.
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**GatewayWorker에서 nginx로 설정한 헤더를 통해 클라이언트 IP를 얻기**

Events.php에 아래 코드를 추가합니다.
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... 다른 코드 생략....
}
```
코드를 추가한 후에 GatewayWorker를 다시 시작해야 합니다.

이제 Events.php의 `onMessage` 및 `onClose` 메서드에서 `$_SESSION['realIP']`를 통해 클라이언트의 실제 IP를 얻을 수 있습니다.

> 참고: Events.php의 `onWorkerStart`, `onConnect`, `onWorkerStop`에서는 `$_SESSION['realIP']`를 직접 사용할 수 없습니다.
