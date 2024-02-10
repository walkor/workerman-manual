# HTTPS 서비스 생성

**질문:**
Workerman은 어떻게 HTTPS 서비스를 만들어 클라이언트가 HTTPS 프로토콜을 사용하여 통신에 연결할 수 있도록 할 수 있나요?

**답변:**
HTTPS 프로토콜은 사실상 HTTP + SSL(SSL)의 조합으로, HTTP 프로토콜에 SSL 계층을 추가한 것입니다. Workerman은 HTTP 프로토콜을 지원하며, SSL(3.3.7 이상의 Workerman 버전이 필요)도 지원합니다. 따라서 HTTP 프로토콜을 기반으로 SSL을 활성화하여 HTTPS 프로토콜을 지원할 수 있습니다.

Workerman에서 HTTPS를 지원하기 위한 두 가지 일반적인 방법이 있으며, Workerman에서 직접 SSL을 활성화하는 방법과 nginx를 사용하여 SSL을 프록시하는 방법이 있습니다. 두 가지 방법 중 하나를 선택하고 동시에 설정하지 않아야 합니다.

## Workerman에서 SSL 활성화

**준비 작업:**

1. Workerman 버전 >= 3.3.7
2. PHP에 openssl 확장이 설치되어 있어야 합니다.
3. /etc/nginx/conf.d/ssl에 인증서 파일(pem/crt 파일 및 키 파일)이 이미 준비되어 있어야 합니다.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // 또는 crt 파일일 수 있습니다.
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // 자체 서명된 인증서인 경우에는 이 옵션을 활성화해야 합니다.
    )
);

$worker = new Worker('http://0.0.0.0:443', $context);
$worker->transport = 'ssl'; // SSL을 활성화하여 HTTP+SSL을 HTTPS로 변환합니다.
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

위의 코드를 사용하여 작업자로 HTTPS 서비스를 생성하고, 클라이언트는 HTTPS 프로토콜을 통해 Workerman에 안전하게 연결할 수 있습니다.

**테스트:**

브라우저 주소 표시줄에 `https://도메인:443`을 입력하여 액세스합니다.

**주의:**

1. HTTPS 포트는 반드시 HTTPS 프로토콜을 사용하여 액세스해야 합니다. HTTP 프로토콜은 액세스할 수 없습니다.
2. 인증서는 일반적으로 도메인과 결합되므로 테스트할 때는 IP 대신 도메인을 사용해야 합니다.
3. HTTPS가 작동하지 않으면 서버 방화벽을 확인해야 합니다.

## nginx를 사용한 SSL 프록시 설정

Workerman 자체의 SSL 대신 nginx를 사용하여 SSL 프록시를 구성하여 HTTPS를 구현할 수도 있습니다.

> **참고**
> nginx 프록시 SSL과 Workerman SSL을 동시에 사용할 수 없으며, 둘 중 하나를 선택해야 합니다.

통신 원리 및 절차는 다음과 같습니다:

1. 클라이언트는 HTTPS 연결을 시작하여 nginx에 연결합니다.
2. nginx는 HTTPS 프로토콜의 데이터를 HTTP 프로토콜로 변환하고 Workerman의 HTTP 포트로 전달합니다.
3. Workerman은 데이터를 받아 비즈니스 로직을 처리한 후 HTTP 프로토콜의 데이터를 nginx에 반환합니다.
4. nginx는 HTTP 프로토콜의 데이터를 다시 HTTPS로 변환하여 클라이언트에 전달합니다.

### nginx 구성 참고
**전제 조건 및 준비 사항:**

1. Workerman이 8181 포트(HTTP 프로토콜)를 수신하도록 가정합니다.
2. 이미 /etc/nginx/conf.d/ssl에 인증서 파일(pem/crt 파일 및 키 파일)이 준비되어 있습니다.
3. nginx가 443 포트에서 WSS 프록시 서비스를 제공하도록 설정하려고 합니다(필요에 따라 포트를 변경할 수 있음).

**다음과 유사한 nginx 구성:**

```nginx
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}

server {
  listen 443;
  server_name site-domain.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location / {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
``` 
