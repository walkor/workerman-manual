# wss 서비스 생성

**질문:**

Workerman에서 어떻게 wss 서비스를 생성하여 클라이언트가 wss 프로토콜을 사용하여 통신을 연결할 수 있도록 할 수 있을까요? 예를 들어 웨이신 미니 프로그램에서 서버에 연결하는 경우입니다.

**답변:**

wss 프로토콜은 사실상 [웹소켓](https://baike.baidu.com/item/WebSocket) + [SSL](https://baike.baidu.com/item/SSL)의 조합입니다. 웹소켓 프로토콜 위에 [SSL](https://baike.baidu.com/item/SSL) 레이어를 추가한 것으로, [HTTPS](https://baike.baidu.com/item/HTTPS)([HTTP](https://baike.baidu.com/item/HTTP) + [SSL](https://baike.baidu.com/item/SSL))와 유사합니다. 그러므로 웹소켓 프로토콜을 기반으로 [SSL](https://baike.baidu.com/item/SSL)을 활성화하여 wss 프로토콜을 지원할 수 있습니다.

## 방법 1: nginx/apache를 이용하여 SSL 프록시 구성 (권장)

**통신 원리 및 절차**

1. 클라이언트가 wss 연결을 nginx/apache에 연결합니다.
2. nginx/apache는 wss 프로토콜 데이터를 ws 프로토콜 데이터로 변환하고 이를 Workerman의 웹소켓 프로토콜 포트로 전달합니다.
3. Workerman은 데이터를 수신한 후 비즈니스 로직 처리를 수행합니다.
4. Workerman이 클라이언트에게 메시지를 보낼 때는 반대의 과정을 거쳐 데이터가 wss 프로토콜로 변환되어 클라이언트에게 전달됩니다.

## nginx 구성 참고
**전제 조건 및 준비 작업:**

1. nginx가 이미 설치되어 있고 버전이 1.3 이상임
2. Workerman이 8282 포트에서 수신 중인 것으로 가정 (웹소켓 프로토콜)
3. 인증서(pem/crt 파일 및 키 파일)은 /etc/nginx/conf.d/ssl 아래에 저장되어 있다고 가정
4. nginx는 일반적으로 웹 서버로 작동하며 다른 서비스도 실행 중입니다. 따라서 기존 사이트 사용에 영향을 주지 않도록 하기 위해 주소 ```domain.com/wss```를 wss 프록시 진입점으로 사용합니다. 즉, 클라이언트 연결 주소는 wss://domain.com/wss가 됩니다.

**nginx 구성 예:**
```nginx
server {
  listen 443;
  # 도메인 구성은 생략...

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
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} - 기타 사이트 구성...
}
```

**테스트**
```javascript
// 인증서는 도메인을 확인합니다. 도메인으로 연결해야 합니다. 포트는 여기에 표시되지 않습니다.
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("연결 성공");
    ws.send('tom');
    alert("서버에 문자열 'tom'을 전송합니다.");
};
ws.onmessage = function(e) {
    alert("서버로부터의 메시지 수신: " + e.data);
};
```

## Apache를 이용한 wss 프록시

Apache를 사용하여 wss를 Workerman에 프록시로 전달할 수도 있습니다.

준비 작업:

1. GatewayWorker가 8282 포트(웹소켓 프로토콜)을 수신 중인 상태
2. SSL 인증서가 이미 /server/httpd/cert/에 저장되어 있음
3. 443 포트를 특정 포트 8282로 전달하기 위해 Apache를 사용
4. httpd-ssl.conf가 로드되어 있음
5. openssl이 설치되어 있음

**proxy_wstunnel_module 모듈 활성화**
```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**SSL 및 프록시 구성**
```apache
#extra/httpd-ssl.conf
DocumentRoot "/웹사이트/디렉토리"
ServerName 도메인

# Proxy Config
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# SSL 프로토콜 지원을 변경하고 보안이 취약한 프로토콜을 제거합니다.
SSLProtocol all -SSLv2 -SSLv3
# 암호화 스위트를 아래와 같이 수정합니다.
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# 인증서 공개 키를 설정합니다.
SSLCertificateFile /server/httpd/cert/your.pem
# 인증서 개인 키를 설정합니다.
SSLCertificateKeyFile /server/httpd/cert/your.key
# 인증서 체인을 설정합니다.
SSLCertificateChainFile /server/httpd/cert/chain.pem
``` 

**테스트**
```javascript
// 인증서는 도메인을 확인합니다. 도메인으로 연결해야 합니다. 포트가 여기에 표시되지 않습니다.
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("연결 성공");
    ws.send('tom');
    alert("서버에 문자열 'tom'을 전송합니다.");
};
ws.onmessage = function(e) {
    alert("서버로부터의 메시지 수신: " + e.data);
};
```

## 방법 2: Workerman에서 SSL 직접 활성화 (비권장)

> **참고:**
> nginx/apache 프록시 SSL 및 Workerman SSL 설정 둘 중 하나만 활성화할 수 있습니다.

**준비 작업:**

1. Workerman 버전이 3.3.7 이상
2. PHP에 openssl 확장이 설치되어 있음
3. 인증서(pem/crt 파일 및 키 파일)이 임의의 디렉토리에 저장되어 있음

**코드:**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 가능한 실제 인증서를 사용하십시오.
$context = array(
    // 더 많은 SSL 옵션은 메뉴얼 참조 바랍니다. http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // 절대 경로를 사용하십시오.
        'local_cert'        => '디스크 경로/server.pem', // crt 파일도 가능합니다.
        'local_pk'          => '디스크 경로/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // 자체 서명 인증서인 경우 필요한 옵션입니다.
    )
);
// 여기서 웹소켓 프로토콜을 설정합니다 (포트는 임의로 설정할 수 있지만 다른 프로그램에서 사용되지 않도록 해야 합니다)
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// transport를 SSL로 설정하여 웹소켓 + SSL인 wss를 활성화합니다.
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

위 코드를 통해 Workerman은 wss 프로토콜을 수신할 수 있게 되며 클라이언트는 wss 프로토콜을 통해 연결하여 안전한 실시간 통신을 구현할 수 있습니다.

**테스트**

Chrome 브라우저를 열고 F12를 눌러 디버그 콘솔을 열어주세요. 콘솔 탭에서 다음 JavaScript 코드를 입력하거나 이 코드를 HTML 페이지에 넣어 실행할 수 있습니다.

```javascript
// 인증서는 도메인을 확인합니다. 도메인으로 연결해야 합니다. 포트는 여기에 표시됩니다.
ws = new WebSocket("wss://domain.com:8282");
ws.onopen = function() {
    alert("연결 성공");
    ws.send('tom');
    alert("서버에 문자열 'tom'을 전송합니다.");
};
ws.onmessage = function(e) {
    alert("서버로부터의 메시지 수신: " + e.data);
};
```

**참고:**

1. 443 포트를 반드시 사용해야 하는 경우에는 위의 첫 번째 방법인 nginx/apache 프록시 방법을 사용하십시오.
2. wss 포트는 wss 프로토콜만 사용할 수 있으며 ws 프로토콜이 wss 포트에 연결할 수 없습니다.
3. 인증서는 일반적으로 도메인에 바인딩되므로 테스트할 때는 클라이언트가 도메인에 연결하고 IP를 사용하지 않도록 하십시오.
4. 연결에 문제가 발생하는 경우 서버 방화벽을 확인하십시오.
5. 이 방법은 PHP 버전 5.6 이상을 필요로 합니다. 왜냐하면 웨이신 미니 프로그램은 tls1.2를 요구하며 PHP 5.6 미만 버전은 tls1.2를 지원하지 않기 때문입니다.

관련 글자:
[프록시를 통해 클라이언트 실제 IP 얻기](https://www.baidu.com/link?url=7YwHtlOqOy2YbP3vFvYUHBzm2h-ZZoASQVSY2VVCpPmrLOiyt32xGEG7LZlLrYftiBvNHxpTlHRvKtUNH5MDyK&wd=&eqid=c2762a700007686b000000025f8d2219)
[Workerman SSL 컨텍스트 옵션 참고](https://php.net/manual/zh/context.ssl.php)
