# 传输加密-ssl/tls

**问：**

如何保证和Workerman之间通讯安全？

**答：**

比较方便的做法是在通讯协议之上增加一层[SSL](https://baike.baidu.com/item/ssl)加密层，比如wss、[https](https://baike.baidu.com/item/https)协议都是基于[SSL](https://baike.baidu.com/item/ssl)加密传输的，非常安全。Workerman自身支持[SSL](https://baike.baidu.com/item/ssl)(```需要Workerman>=3.3.7```)，只需要设置下属性即可开启SSL。

当然开发者也可以基于某些加解密算法实现一套自己的加解密机制。

## Workerman开启ssl方法如下：


**准备工作：**

1、Workerman版本不小于3.3.7

2、PHP安装了openssl扩展

3、已经申请了证书（pem/crt文件及key文件）放在了/etc/nginx/conf.d/ssl下

**代码：**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 证书最好是申请的证书
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // 也可以是crt文件
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, //如果是自签名证书需要开启此选项
    )
);
// 这里设置的是websocket协议，也可以http协议或者其它协议
$worker = new Worker('websocket://0.0.0.0:443', $context);
// 设置transport开启ssl
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Workerman开启服务器名称指示 [SNI（Server Name Indication）](https://baike.baidu.com/item/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA)
可实现在同一IP、端口情况下，绑定多个证书。

**合并证书.pem和.key文件：**

将每个证书的.pem和对应的.key文件内容合并，将.key文件内容添加到.pem文件结尾。（若.pem文件内已包含私钥，则可忽略。）

**请注意是单个证书，不是把所有证书复制到一个文件**

例如*host1.com.pem*合并后的pem文件内容大概如下：
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

**代码：**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // 开启SNI
        'SNI_server_certs' => [ // 设置多个证书
            'host1.com' => '/path/host1.com.pem', // 证书1
            'host2.com' => '/path/host2.com.pem', // 证书2
        ],
    )
);
$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
