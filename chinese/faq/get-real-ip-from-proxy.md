## 透过nginx/apache代理如何获取客户端真实ip ?
使用nginx/apache作为workerman代理，nginx/apache实际上充当了workerman的客户端，所以在workerman上获取的客户端ip为nginx/apache服务器的ip，并非实际的客户端ip。如何获取客户端真实ip可以参考下面的方法。

**原理：**

nginx/apache将客户端真实ip通过http header传递进来，例如nginx配置中location里的加上```proxy_set_header X-Real-IP $remote_addr;```设置。workerman通过读取这个header值，将此值保存到```$connection对象里```，(GatewayWorker可以保存到```$_SESSION```变量里)，使用的时候直接读取变量即可。

**注意：**

以下配置适用于http/https ws/wss协议。其它协议要获取客户端ip方法类似，需要代理服务器在数据包插入一段ip数据透传真实客户端ip。

**nginx配置类似如下**：
```
server {
  listen 443;

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
    # 这部分是利用http头透传真实客户端ip
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} 站点的其它配置...
}
```

**workerman从nginx设置的header里读取客户端ip**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// 客户端练上来时，即完成TCP三次握手后的回调
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * 客户端websocket握手时的回调onWebSocketConnect
    * 在onWebSocketConnect回调中获得nginx通过http头中的X_REAL_IP值
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * connection对象本没有realIP属性，这里给connection对象动态添加个realIP属性
        * 记住php对象是可以动态添加属性的，你也可以用自己喜欢的属性名
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 当使用客户端真实ip时，直接使用$connection->realIP即可
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**GatewayWorker从nginx设置的header里获取客户端ip**

在Events.php加上下面的代码
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... 省略其它代码....
}
```
代码加完后需要重启GatewayWorker。

这样就可以在Events.php中的`onMessage` 和`onClose`方法中通过`$_SESSION['realIP']`得到客户端的真实ip了。

> 注意：Events.php中`onWorkerStart` `onConnect` `onWorkerStop` 无法直接使用`$_SESSION['realIP']`。
