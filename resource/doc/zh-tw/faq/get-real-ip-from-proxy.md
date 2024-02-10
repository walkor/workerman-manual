## 透過nginx/apache代理如何獲取客戶端真實IP？
當使用nginx/apache作為workerman的代理時，nginx/apache實際上充當了workerman的客戶端，因此在workerman上獲取的是nginx/apache伺服器的IP，而非實際客戶端的IP。要獲取客戶端真實IP可以參考下面的方法。

**原理：**

nginx/apache會通過http header將客戶端的真實IP傳遞過來，例如在nginx配置的location內加上```proxy_set_header X-Real-IP $remote_addr;```的設置。Workerman通過讀取這個header值，將其保存在```$connection對象```內（GatewayWorker可以保存在```$_SESSION```變數內），使用時直接讀取即可。

**注意：**

以下配置適用於http/https ws/wss協議。其他協議獲取客戶端IP的方法類似，需要代理伺服器在資料包中插入一段IP數據以透傳真實客戶端IP。

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
    # 這部分是利用http頭透傳真實客戶端IP
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} 站點的其他配置...
}
```

**workerman從nginx設置的header裡讀取客戶端IP**
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
    * 在onWebSocketConnect回調中獲得nginx通過http頭中的X_REAL_IP值
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * connection對象本沒有realIP屬性，這裡給connection對象動態添加個realIP屬性
        * 記住php對象是可以動態添加屬性的，你也可以用自己喜歡的屬性名
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

**GatewayWorker從nginx設置的header裡獲取客戶端IP**
在Events.php加上以下的代碼
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... 省略其它代碼....
}
```
代碼加完後需要重啟GatewayWorker。

這樣就可以在Events.php的`onMessage` 和`onClose`方法中通過`$_SESSION['realIP']`得到客戶端的真實IP了。

> 注意：Events.php中的`onWorkerStart` `onConnect` `onWorkerStop` 無法直接使用`$_SESSION['realIP']`。
