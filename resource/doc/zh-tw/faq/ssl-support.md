# 傳輸加密-ssl/tls

**問：**

如何確保和Workerman之間通訊安全？

**答：**

比較方便的做法是在通訊協議之上增加一層[SSL](https://baike.baidu.com/item/ssl)加密層，比如wss、[https](https://baike.baidu.com/item/https)協議都是基於[SSL](https://baike.baidu.com/item/ssl)加密傳輸的，非常安全。Workerman自身支持[SSL](https://baike.baidu.com/item/ssl)(```需要Workerman>=3.3.7```)，只需要設置下屬性即可開啟SSL。

當然開發者也可以基於某些加解密算法實現一套自己的加解密機制。

## Workerman開啟ssl方法如下：

**準備工作：**

1、Workerman版本不小於3.3.7

2、PHP安裝了openssl擴展

3、已經申請了證書（pem/crt文件及key文件）放在了/etc/nginx/conf.d/ssl下

**代碼：**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 證書最好是申請的證書
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // 也可以是crt文件
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, //如果是自簽名證書需要開啟此選項
    )
);
// 這裡設置的是websocket協議，也可以http協議或者其它協議
$worker = new Worker('websocket://0.0.0.0:443', $context);
// 設置transport開啟ssl
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Workerman開啟伺服器名稱指示 [SNI（Server Name Indication）](https://baike.baidu.com/item/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA)
可實現在同一IP、端口情況下，綁定多個證書。

**合併證書.pem和.key文件：**

將每個證書的.pem和對應的.key文件內容合併，將.key文件內容添加到.pem文件結尾。（若.pem文件內已包含私鑰，則可忽略。）

**請注意是單個證書，不是把所有證書複製到一個文件** 

例如*host1.com.pem*合併後的pem文件內容大概如下：

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

**代碼：**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // 開啟SNI
        'SNI_server_certs' => [ // 設置多個證書
            'host1.com' => '/path/host1.com.pem', // 证书1
            'host2.com' => '/path/host2.com.pem', // 证书2
        ],
        'local_cert' => '/path/default.com.pem', // 默認證書
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
