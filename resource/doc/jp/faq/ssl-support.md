# トランスポート暗号化 - SSL/TLS

**質問：**
Workermanとの通信をどのようにセキュアにするのですか？

**回答：**
比較的簡単な方法は、通信プロトコルの上にSSL暗号化レイヤーを追加することです。たとえば、wss、httpsプロトコルはすべてSSL暗号化伝送に基づいており、非常に安全です。Workerman自体はSSL（```Workerman>=3.3.7```が必要）をサポートしており、プロパティを設定するだけでSSLを有効にできます。

もちろん、開発者は暗号化／復号化アルゴリズムに基づいて独自の暗号化／復号化メカニズムを実装することもできます。

## WorkermanでSSLを有効にする方法は以下の通りです：

**事前準備:**

1. Workermanバージョンが3.3.7以上であること
2. PHPにopenssl拡張機能がインストールされていること
3. 証明書（pem/crtファイルとキーファイル）を既に取得し、/etc/nginx/conf.d/sslに保存されていること

**コード:**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 証明書はできるだけ取得した方が良い
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem',
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true,
    )
);
// ここでwebsocketプロトコルを設定し、httpプロトコルまたはその他のプロトコルも設定できます
$worker = new Worker('websocket://0.0.0.0:443', $context);
// トランスポートをSSLに設定
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Workermanでサーバ名指示（SNI：Server Name Indication）を有効にすることで、同じIPとポートに複数の証明書をバインドできます。

**証明書.pemと.keyファイルを結合する：**
各証明書の.pemファイルと対応する.keyファイルの内容を結合し、.keyファイルの内容を.pemファイルの末尾に追加します。（.pemファイルに秘密鍵が含まれている場合は無視して構いません。）

**1つの証明書、すべての証明書を1つのファイルにコピーするわけではありません**

例えば*host1.com.pem*を結合した後のpemファイルの内容は以下のようになります：
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

**コード：**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // SNIを有効にする
        'SNI_server_certs' => [ // 複数の証明書を設定
            'host1.com' => '/path/host1.com.pem', // 証明書1
            'host2.com' => '/path/host2.com.pem', // 証明書2
        ],
        'local_cert' => '/path/default.com.pem', // デフォルトの証明書
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
