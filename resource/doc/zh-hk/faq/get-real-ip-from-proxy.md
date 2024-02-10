## 透過nginx/apache代理如何獲取客戶端真實IP？
當使用nginx/apache作為workerman的代理時，nginx/apache實際上充當了workerman的客戶端，因此在workerman上獲取的客戶端IP是nginx/apache伺服器的IP，而非實際的客戶端IP。要獲取客戶端真實IP，可以參考以下方法。

**原理：**

nginx/apache將客戶端真實IP通過HTTP標頭傳遞過來，例如nginx配置中在location中加上```proxy_set_header X-Real-IP $remote_addr;```進行設置。workerman通過讀取這個標頭值，將此值保存到```$connection對象```中，(GatewayWorker可以保存到```$_SESSION```變量中)，在使用時直接讀取該變量即可。

**注意：**

以下配置適用於HTTP/HTTPS WS/WSS協議。其他協議要獲取客戶端IP的方法類似，需要代理伺服器在數據包中插入一段IP數據，透傳真實客戶端IP。

**nginx配置類似如下**：
```nginx
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
    # 透過HTTP標頭透傳真實客戶端IP
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} 站點的其他配置...
}
```

**workerman從nginx設置的標頭中讀取客戶端IP**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// 客戶端連上來時，即完成TCP三次握手後的回調
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * 客戶端websocket握手時的回調onWebSocketConnect
    * 在onWebSocketConnect回調中獲得nginx通過HTTP標頭中的X_REAL_IP值
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * connection對象本沒有realIP屬性，這裡給connection對象動態添加個realIP屬性
        * 記住PHP對象是可以動態添加屬性的，你也可以用自己喜歡的屬性名
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 當使用客戶端真實IP時，直接使用$connection->realIP即可
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**GatewayWorker從nginx設置的標頭中獲取客戶端IP**

在Events.php裡加上下面的代碼
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... 省略其他代碼....
}
```
代碼加完後需要重啟GatewayWorker。

這樣就可以在Events.php中的`onMessage` 和`onClose`方法中通過`$_SESSION['realIP']`得到客戶端的真實IP了。

> 注意：Events.php中的`onWorkerStart` `onConnect` `onWorkerStop` 無法直接使用`$_SESSION['realIP']`。
