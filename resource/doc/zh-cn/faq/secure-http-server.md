# 创建https服务

**问：**

Workerman如何创建一个[https](https://baike.baidu.com/item/https)服务，使得客户端可以用过[https](https://baike.baidu.com/item/https)协来连接通讯。


**答：**

[https](https://baike.baidu.com/item/https)协议实际是[http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl)，就是在[http](https://baike.baidu.com/item/http)协议上加入[SSL](https://baike.baidu.com/item/ssl)层。Workerman支持[http](https://baike.baidu.com/item/http)协议，同时也支持[SSL](https://baike.baidu.com/item/ssl)(```需要Workerman版本>=3.3.7```)，
所以只需要在[http](https://baike.baidu.com/item/http)协议的基础上开启[SSL](https://baike.baidu.com/item/ssl)即可支持[https](https://baike.baidu.com/item/https)协议。

让workerman支持https有两种通用方案，一种是workerman直接开启SSL，另外一种是用nginx代理SSL。两种方案选其一即可，不可同时设置。

## Workerman开启SSL

**准备工作：**

1、Workerman版本>=3.3.7

2、PHP安装了openssl扩展

3、已经申请了证书（pem/crt文件及key文件）放在了/etc/nginx/conf.d/ssl下

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
// 这里设置的是http协议
$worker = new Worker('http://0.0.0.0:443', $context);
// 设置transport开启ssl，变成http+SSL即https
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

通过Workerman以上的代码就创建了https服务，客户端就可以通过https协议来连接workerman实现安全加密通讯了。

**测试：**

浏览器地址栏输入```https://域名:443```访问。

**注意：**

1、https端口必须用https协议访问，http协议无法访问。

2、证书一般是与域名绑定的，所以测试的时候请使用域名，不要使用ip。

3、如果使用https无法访问请检查服务器防火墙。


## 利用nginx作为ssl的代理

除了用Workerman自身的SSL，也可以利用nginx作为SSL代理实现https。

> **注意**
> nginx代理SSL和Workerman设置SSL二选一，不能同时开启。

通讯原理及流程是：

1、客户端发起https连接连到nginx

2、nginx将https协议的数据转换成http协议并转发到Workerman的http端口

3、Workerman收到数据后做业务逻辑处理，返回http协议的数据给nginx

4、nginx再将http协议的数据转换成https，转发给客户端


### nginx配置参考
**前提条件及准备工作：**

1、假设Workerman监听的是8181端口(http协议)

2、已经申请了证书（pem/crt文件及key文件）放在了/etc/nginx/conf.d/ssl下

3、打算利用nginx开启443端口对外提供wss代理服务（端口可以根据需要修改）

**nginx配置类似如下**：

```

upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}


server {
  listen 443;
  server_name 站点域名.com;
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

