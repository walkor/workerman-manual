# wssサービスの作成

**質問：**

Workermanでは、クライアントがwssプロトコルを使用して通信するために、wssサービスを作成する方法は何ですか？例えば、WeChatの小プログラム内でサーバーに接続するためです。

**回答：**

実際のところ、wssプロトコルは[WebSocket](https://baike.baidu.com/item/WebSocket)+[SSL](https://baike.baidu.com/item/ssl)の組み合わせであり、つまりWebSocketプロトコルにSSL層を追加したもので、これは[https](https://baike.baidu.com/item/https)([http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl))に似ています。
したがって、WebSocketプロトコルの基礎上でSSLを有効にするだけで、wssプロトコルをサポートできます。

## 方法1：nginx/apacheを使用したSSLプロキシ（推奨）

**通信原理とフロー**

1. クライアントがwss接続をnginx/apacheにリクエスト
2. nginx/apacheはwssプロトコルのデータをwsプロトコルのデータに変換し、WorkermanのWebSocketプロトコルポートに転送
3. Workermanはデータを受信し、ビジネスロジックを処理
4. Workermanがクライアントにメッセージを送信する場合は、逆のプロセスが行われ、データはnginx/apacheによってwssプロトコルに変換されてクライアントに送信されます。

## nginxの設定例

**前提条件と準備**

1. nginxがインストールされており、バージョンが1.3以上であること
2. Workermanが8282ポート（WebSocketプロトコル）をリッスンしていると仮定
3. 証明書（pem/crtファイルとキー）がすでに取得されている（/etc/nginx/conf.d/sslなどのパスに保存されていると仮定）
4. nginxをウェブサイトサーバーとして使用している場合、元のサイトの使用に影響を与えないようにするために、アドレス ```domain.com/wss``` をwssのプロキシエントリーポイントとして使用することを計画しています。つまり、クライアントの接続アドレスは wss://domain.com/wss になります。

**nginxの設定例：**
```nginx
server {
  listen 443;
  # ドメインの設定は省略...

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
  
  # location / {} その他のサイトの設定...
}
```

**テスト**
```javascript
// 証明書はドメインをチェックするために使用されます。ドメインで接続してください。ポートは記述しないでください
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("接続しました");
    ws.send('tom');
    alert("サーバーに文字列'tom'を送信します");
};
ws.onmessage = function(e) {
    alert("サーバーからのメッセージを受信しました：" + e.data);
};
```

## Apacheを使用したwssプロキシも可能です

Apacheを使用して443ポートを指定のポート8282に転送することもできます。

**準備**

1. GatewayWorkerが8282ポート（WebSocketプロトコル）をリッスンしていると仮定
2. ssl証明書がすでに/server/httpd/cert/に保存されていると仮定
3. apacheで443ポートを指定のポートに転送するために、httpd-ssl.confがロードされていると仮定
4. opensslがインストールされていること

**proxy_wstunnel_module モジュールの有効化**
```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**SSLおよびプロキシの設定**
```apache
#extra/httpd-ssl.conf
DocumentRoot "/web/directory"  # ウェブサイトのディレクトリ
ServerName domain  # ドメイン

# Proxy Config
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# SSLプロトコルのサポートを追加し、安全でないプロトコルを削除
SSLProtocol all -SSLv2 -SSLv3
# 暗号スイートを変更
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# 証明書公開鍵の設定
SSLCertificateFile /server/httpd/cert/your.pem
# 証明書秘密鍵の設定
SSLCertificateKeyFile /server/httpd/cert/your.key
# 証明書チェーンの設定
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**テスト**
```javascript
// 証明書はドメインをチェックするために使用されます。ドメインで接続してください。ポートは記述しないでください
ws = new WebSocket("wss://domain.com:443/wss");

ws.onopen = function() {
    alert("接続しました");
    ws.send('tom');
    alert("サーバーに文字列'tom'を送信します");
};
ws.onmessage = function(e) {
    alert("サーバーからのメッセージを受信しました：" + e.data);
};
```

## 方法2：Workermanで直接SSLを有効化する（推奨されない）

> **注意**
> nginx/apacheを使用したSSLプロキシとWorkermanでのSSL設定はどちらかを選択する必要があります。両方を同時に使用することはできません。

**準備**

1. Workermanバージョン>=3.3.7
2. PHPにopenssl拡張がインストールされていること
3. 証明書（pem/crtファイルとキー）がディスク上の任意のディレクトリに保存されていること

**コード：**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 証明書はなるべく申請した方が良い
$context = array(
    // 追加のsslオプションについては、マニュアルを参照してください http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // 絶対パスを使用してください
        'local_cert'        => 'ディスク上のパス/server.pem',  // crtファイルでも構いません
        'local_pk'          => 'ディスク上のパス/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, //自己署名証明書の場合、このオプションを有効にする必要があります
    )
);
// ここでwebsocketプロトコルを設定します（ポートは任意ですが、他のプログラムが使用していないことを確認してください）
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// transportを使用してsslを有効にします（websocket+sslはwssに対応します）
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

上記のコードにより、Workermanはwssプロトコルをリッスンし、クライアントはwssプロトコルを使用してWorkermanに安全なリアルタイム通信を実現することができます。

**テスト**

Google Chromeを開き、F12を押してデバッグコンソールを開きます。コンソールに以下のコードを入力するか、HTMLページに以下のコードを配置して実行します。

```javascript
// 証明書はドメインをチェックするために使用されます。ドメインで接続してください。ここでポート番号があります。
ws = new WebSocket("wss://domain.com:8282");
ws.onopen = function() {
    alert("接続しました");
    ws.send('tom');
    alert("サーバーに文字列'tom'を送信します");
};
ws.onmessage = function(e) {
    alert("サーバーからのメッセージを受信しました：" + e.data);
};
```

**注意：**

1. 必ず443ポートを使用する必要がある場合は、方法1のnginx/apacheプロキシを使用してwssを実装してください。
2. wssポートにはwssプロトコルでのみアクセスできます。wsではアクセスできません。
3. 証明書は通常、ドメインにバインドされているため、テストする際はクライアントがドメインを使用して接続し、IPアドレスではなく、使用しないでください。
4. アクセスできない場合は、サーバーのファイアウォールをチェックしてください。
5. この方法を使用するには、PHPのバージョンが5.6以上である必要があります。なぜなら、WeChatの小プログラムがtls1.2を要求するためで、PHP5.6未満のバージョンはtls1.2をサポートしていないからです。

関連記事：  
[プロキシからクライアントの実際のIPを取得](get-real-ip-from-proxy.md)  
[WorkermanのSSLコンテキストオプションのリファレンス](https://php.net/manual/zh/context.ssl.php)
