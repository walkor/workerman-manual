# 보안 전송 - SSL/TLS

**질문:**

Workerman과의 통신을 어떻게 보안할 수 있습니까?

**답변:**

가장 편리한 방법은 통신 프로토콜 상위에 [SSL](https://baike.baidu.com/item/ssl) 암호화 계층을 추가하는 것입니다. 예를 들어 wss, [https](https://baike.baidu.com/item/https) 프로토콜은 모두 [SSL](https://baike.baidu.com/item/ssl) 암호화 전송을 기반으로 하므로 매우 안전합니다. Workerman은 본인이 [SSL](https://baike.baidu.com/item/ssl)을 지원합니다(```Workerman>=3.3.7``` 버전 이상 필요), 속성을 설정하여 SSL을 쉽게 활성화할 수 있습니다.

물론, 개발자는 어떤 암호화/복호화 알고리즘을 기반으로 자체 암호화/복호화 메커니즘을 구현할 수도 있습니다.

## Workerman에서 SSL 활성화 방법은 다음과 같습니다:

**준비 작업:**

1. Workerman 버전은 3.3.7 이상이어야 합니다.
2. PHP에 openssl 확장이 설치되어 있어야 합니다.
3. 인증서(pem/crt 파일 및 키 파일)가 /etc/nginx/conf.d/ssl에 저장되어 있어야 합니다.

**코드:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 인증서는 최상의 인증서를 신청하는 것이 좋습니다
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // 또는 crt 파일
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // 자체 서명된 인증서인 경우 이 옵션을 활성화해야 합니다
    )
);
// 여기서는 웹소켓 프로토콜을 설정했지만, http 프로토콜 또는 기타 프로토콜도 가능합니다
$worker = new Worker('websocket://0.0.0.0:443', $context);
// transport를 ssl로 설정
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Workerman에서 서버 이름 지시 [SNI (Server Name Indication)](https://baike.baidu.com/item/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA) 활성화
동일한 IP와 포트에서 여러 인증서를 바인딩할 수 있도록 합니다.

**.pem 및 .key 파일 병합:**

각 인증서의 .pem 및 해당 .key 파일의 내용을 병합하고, .key 파일의 내용을 .pem 파일 끝에 추가합니다.(.pem 파일 내에서 이미 개인 키가 포함되어 있는 경우 무시할 수 있습니다.)

**개별 인증서를 한 파일에 모두 복사하지 말아야 함을 주의하세요.**

예를 들어 *host1.com.pem*을 병합한 후의 pem 파일 내용은 대략 다음과 같을 것입니다:
```text
-----BEGIN CERTIFICATE-----
MIIGXTCBA...
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIFBzCCA...
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAA....
-----END RSA PRIVATE KEY-----
```

**코드:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // SNI 활성화
        'SNI_server_certs' => [ // 여러 인증서 설정
            'host1.com' => '/path/host1.com.pem', // 인증서 1
            'host2.com' => '/path/host2.com.pem', // 인증서 2
        ],
        'local_cert' => '/path/default.com.pem', // 기본 인증서
        'local_pk'   => '/path/default.com.key',
    )
);
$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
