# HTTPSサービスの作成

**質問：**

Workermanは、クライアントがHTTPSプロトコルを使用して接続できるように、HTTPSサービスをどのように作成しますか？


**回答：**

HTTPSプロトコルは実際にはHTTPにSSLレイヤーを追加したものであり、WorkermanはHTTPプロトコルをサポートすると同時にSSL（Workermanのバージョンが3.3.7以上である必要があります）もサポートしています。したがって、HTTPプロトコルをベースにSSLを有効にすれば、HTTPSプロトコルをサポートすることができます。

WorkermanをHTTPSをサポートするための2つの一般的なアプローチがあります。1つはWorkermanが直接SSLを開始することであり、もう1つはnginxを使用してSSLをプロキシすることです。これらのアプローチのどちらかを選択し、同時に設定を行うことはできません。

## WorkermanでSSLを開始する

**準備：**

1. Workermanのバージョンが3.3.7以上であること。
2. PHPにopenssl拡張機能がインストールされていること。
3. 証明書（pem/crtファイルおよびキーファイル）が既に/etc/nginx/conf.d/sslに配置されていること。

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 証明書は申請済みのものを使用することが望ましい
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // またはcrtファイル
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // セルフサイン証明書を使用する場合はこのオプションを有効にする
    )
);
// ここではhttpプロトコルを設定しています
$worker = new Worker('http://0.0.0.0:443', $context);
// transportをsslに設定し、http+SSL（https）にする
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

上記のコードを使用して、WorkermanでHTTPSサービスを作成することができます。クライアントはこれによって安全な暗号化通信を実現するためにHTTPSプロトコルを使用してWorkermanに接続することができます。

**テスト：**

ブラウザのアドレスバーに「https://ドメイン名:443」と入力してアクセスします。

**注意：**

1. HTTPSポートはhttpsプロトコルを使用して接続する必要があります。httpプロトコルではアクセスできません。
2. 証明書は通常、ドメインにバインドされているため、テストする際はIPアドレスではなく、ドメインを使用してください。
3. HTTPSが機能しない場合は、サーバーのファイアウォールを確認してください。

## nginxを使用してSSLプロキシを設定する

WorkermanのSSLとは別に、nginxを使用してSSLプロキシを実装することもできます。

> **注意**
> nginxプロキシSSLとWorkermanのSSL設定は相互に排他的であり、同時に有効にすることはできません。

通信の原理と手順は次のとおりです：

1. クライアントがhttps接続をnginxにリクエストします。
2. nginxはhttpsプロトコルのデータをhttpプロトコルに変換し、Workermanのhttpポートに転送します。
3. Workermanはデータを受信し、ビジネスロジックを処理してhttpプロトコルのデータをnginxに返します。
4. nginxはhttpプロトコルのデータをhttpsに変換し、クライアントに転送します。

### nginxの設定例

**前提条件と準備：**

1. Workermanが8181ポート（httpプロトコル）をリッスンしているとします。
2. 証明書（pem/crtファイルおよびキーファイル）が既に/etc/nginx/conf.d/sslに配置されていること。
3. nginxを使用して443ポートでwssプロキシサービスを提供する意図があること（ポートは必要に応じて変更できます）。

**nginxの設定は次のようになります**：

```

upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}


server {
  listen 443;
  server_name サイトのドメイン.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /
  {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
