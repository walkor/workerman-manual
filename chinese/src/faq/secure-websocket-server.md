# 创建wss服务

**问：**

Workerman如何创建一个wss服务，使得客户端可以用过wss协来连接通讯，比如在微信小程序中连接服务端。


**答：**

可以利用nginx作为ssl的代理。通讯原理及流程是：

1、客户端发起wss连接连到nginx

2、nginx将wss协议的数据转换成ws协议并转发到Workerman

3、Workerman收到数据后做业务逻辑处理

4、Workerman给客户端发送消息时，则是相反的过程，数据经过nginx转换成wss协议然后发给客户端


## nginx配置参考
**前提条件及准备工作：**

1、假设Workerman监听的是8282端口(websocket协议)

2、已经申请了证书（pem/crt文件及key文件）放在了/etc/nginx/conf.d/ssl下

3、打算利用nginx开启4431端口对外提供wss代理服务（端口可以根据需要修改）

**nginx配置类似如下**：

```
server {
  listen 4431;

  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/laychat/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /
  {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header X-Real-IP $remote_addr;
  }
}
```

**测试**

打开chrome浏览器，按F12打开调试控制台，在Console一栏输入(或者把下面代码放入到html页面用js运行)

```javascript
// 证书是会检查域名的，请使用域名连接
ws = new WebSocket("wss://域名:4431");
ws.onopen = function() {
    alert("连接成功");
    ws.send('tom');
    alert("给服务端发送一个字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

``` 注意：如果出现无法访问的情况，请检查服务器防火墙。 ```

## 透过nginx wss代理如何获取客户端真实ip ?
使用nginx作为wss代理，nginx实际上充当了workerman的客户端，所以在workerman上获取的客户端ip为nginx服务器的ip，并非实际的客户端ip。如何获取客户端真实ip可以参考下面的方法。

**原理：**

nginx将客户端真实ip通过http header传递进来，即上面nginx配置中location里的```proxy_set_header X-Real-IP $remote_addr;```设置。workerman通过读取这个header值，将此值保存到```$connection对象里```，(GatewayWorker可以保存到```$_SESSION```变量里)，使用的时候直接读取变量即可。

**workerman从nginx设置的header里读取客户端ip**

```php
<?php
require_once __DIR__ . '/../Workerman/Autoloader.php';
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:7272');

// 客户端练上来时，即完成TCP三次握手后的回调
$worker->onConnect = function($connection) {
   /**
    * 客户端websocket握手时的回调onWebSocketConnect
    * 在onWebSocketConnect回调中获得nginx通过http头中的X_REAL_IP值
    */
   $connection->onWebSocketConnect = function($connection){
       /**
        * connection对象本没有realIP属性，这里给connection对象动态添加个realIP属性
        * 记住php对象是可以动态添加属性的，你也可以用自己喜欢的属性名
        *、
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function($connection, $data)
{
    // 当使用客户端真实ip时，直接使用$connection->realIP即可
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**GatewayWorker从nginx设置的header里获取客户端ip**

在start_gateway.php加上下面的代码
```php
$gateway->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $http_header)
    {
        $_SESSION['realIP'] = $_SERVER['HTTP_X_REAL_IP'];
    };
};
```
代码加完后需要重启GatewayWorker。

这样就可以在Events.php中通过```$_SESSION['realIP']```得到客户端的真实ip了
