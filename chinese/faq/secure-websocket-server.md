# 创建wss服务

**问：**

Workerman如何创建一个wss服务，使得客户端可以用过wss协来连接通讯，比如在微信小程序中连接服务端。


**答：**

wss协议实际是[websocket](https://baike.baidu.com/item/WebSocket)+[SSL](https://baike.baidu.com/item/ssl)，就是在websocket协议上加入[SSL](https://baike.baidu.com/item/ssl)层，类似[https](https://baike.baidu.com/item/https)([http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl))。
所以只需要在[websocket](https://baike.baidu.com/item/WebSocket)协议的基础上开启[SSL](https://baike.baidu.com/item/ssl)即可支持wss协议。

## 方法一、利用nginx/apache代理SSL(推荐)


**通讯原理及流程**

1、客户端发起wss连接连到nginx/apache

2、nginx/apache将wss协议的数据转换成ws协议数据并转发到Workerman的websocket协议端口

3、Workerman收到数据后做业务逻辑处理

4、Workerman给客户端发送消息时，则是相反的过程，数据经过nginx/apache转换成wss协议然后发给客户端


## nginx配置参考
**前提条件及准备工作：**

1、已经安装nginx，版本不低于1.3

2、假设Workerman监听的是8282端口(websocket协议)

3、已经申请了证书（pem/crt文件及key文件）假设放在了/etc/nginx/conf.d/ssl下

4、打算利用nginx开启443端口对外提供wss代理服务（端口可以根据需要修改）

5、nginx一般作为网站服务器运行着其它服务，为了不影响原来的站点使用，这里使用地址 ```域名.com/wss``` 作为wss的代理入口。也就是客户端连接地址为 wss://域名.com/wss

**nginx配置类似如下**：
```
server {
  listen 443;
  # 域名配置省略...

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
  
  # location / {} 站点的其它配置...
}
```
**测试**
```javascript
// 证书是会检查域名的，请使用域名连接。注意这里不写端口
ws = new WebSocket("wss://域名.com/wss");

ws.onopen = function() {
    alert("连接成功");
    ws.send('tom');
    alert("给服务端发送一个字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

## 利用apache代理wss

也可以利用apache作为wss代理转发给workerman。

准备工作：

1、GatewayWorker 监听 8282 端口(websocket协议)

2、已经申请了ssl证书, 假设放在了/server/httpd/cert/ 下

3、利用apache转发443端口至指定端口8282

4、httpd-ssl.conf 已加载

5、openssl 已安装

**启用 proxy_wstunnel_module 模块**
```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**配置SSL及代理**
```
#extra/httpd-ssl.conf
DocumentRoot "/网站/目录"
ServerName 域名

# Proxy Config
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# 添加 SSL 协议支持协议,去掉不安全的协议
SSLProtocol all -SSLv2 -SSLv3
# 修改加密套件如下
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# 证书公钥配置
SSLCertificateFile /server/httpd/cert/your.pem
# 证书私钥配置
SSLCertificateKeyFile /server/httpd/cert/your.key
# 证书链配置,
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**测试**
```javascript
// 证书是会检查域名的，请使用域名连接。注意没有端口
ws = new WebSocket("wss://域名.com/wss");

ws.onopen = function() {
    alert("连接成功");
    ws.send('tom');
    alert("给服务端发送一个字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```


## 方法二 ，直接用Workerman开启SSL(不推荐)

> **注意**
> nginx/apache代理SSL和Workerman设置SSL二选一，不能同时开启。

**准备工作：**

1、Workerman版本>=3.3.7

2、PHP安装了openssl扩展

3、已经申请了证书（pem/crt文件及key文件）放在磁盘任意目录

**代码：**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 证书最好是申请的证书
$context = array(
    // 更多ssl选项请参考手册 http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // 请使用绝对路径
        'local_cert'        => '磁盘路径/server.pem', // 也可以是crt文件
        'local_pk'          => '磁盘路径/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, //如果是自签名证书需要开启此选项
    )
);
// 这里设置的是websocket协议（端口任意，但是需要保证没被其它程序占用）
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// 设置transport开启ssl，websocket+ssl即wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

通过以上的代码，Workerman就监听了wss协议，客户端就可以通过wss协议来连接workerman实现安全即时通讯了。

**测试**

打开chrome浏览器，按F12打开调试控制台，在Console一栏输入(或者把下面代码放入到html页面用js运行)

```javascript
// 证书是会检查域名的，请使用域名连接，注意这里有端口号
ws = new WebSocket("wss://域名.com:8282");
ws.onopen = function() {
    alert("连接成功");
    ws.send('tom');
    alert("给服务端发送一个字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

**注意：**

1、如果必须使用443端口请使用上面第一种方案nginx/apache代理方式实现wss。

2、wss端口只能通过wss协议访问，ws无法访问wss端口。

3、证书一般是与域名绑定的，所以测试的时候客户端请使用域名连接，不要使用ip去连。

4、如果出现无法访问的情况，请检查服务器防火墙。

5、此方法要求PHP版本>=5.6，因为微信小程序要求tls1.2，而PHP5.6以下版本不支持tls1.2。



相关文章：  
[透过代理获取客户端真实ip](get-real-ip-from-proxy.md)  
[workerman的ssl上下文选项参考](https://php.net/manual/zh/context.ssl.php)


