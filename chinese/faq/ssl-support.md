# 传输加密-ssl/tsl

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
        'local_cert'  => '/etc/nginx/conf.d/ssl/server.pem', // 也可以是crt文件
        'local_pk'    => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer' => false,
    )
);
// 这里设置的是websocket协议，也可以http协议或者其它协议
$worker = new Worker('http://0.0.0.0:443', $context);
// 设置transport开启ssl
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
