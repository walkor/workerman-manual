## Nginx / Apacheプロキシを介してクライアントの実際のIPアドレスを取得する方法は？

WorkermanプロキシとしてNginx / Apacheを使用すると、実際にはNginx / ApacheがWorkermanのクライアントとして機能し、したがってWorkermanで取得されるクライアントIPは実際のクライアントIPではなく、Nginx / ApacheサーバーのIPになります。クライアントの実際のIPを取得する方法は以下の方法を参照してください。

**原理：**

Nginx / Apacheはクライアントの実際のIPをHTTPヘッダを介して転送します。たとえば、Nginxの設定でlocation内に```proxy_set_header X-Real-IP $remote_addr;```を追加します。Workermanはこのヘッダの値を読み取り、この値を```$connectionオブジェクト```に保存します。(GatewayWorkerでは```$_SESSION```変数に保存されます)。使用時には、変数を直接読み取ればよいです。

**注意：**

以下の設定はhttp/https ws/wssプロトコルに適用されます。他のプロトコルの場合も、クライアントのIPを取得する方法は同様です。ただし、プロキシサーバーが実際のクライアントIPを透過的にデータパケットに挿入する必要があります。

**Nginxの設定の例**:
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
    # ここでHTTPヘッダを使用して実際のクライアントIPを透過する
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} 他のサイトの設定...
}
```

**WorkermanがNginxの設定からヘッダーを取得する方法**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// クライアントが接続すると、TCPの三方ハンドシェイクが完了すると次のコールバックが実行されます
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * クライアントのWebSocket接続時のコールバックonWebSocketConnect
    * onWebSocketConnectコールバック内でNginxからのHTTPヘッダーX_REAL_IPの値を取得します
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * connectionオブジェクトにはrealIPプロパティは元々ありませんが、ここでconnectionオブジェクトに動的にrealIPプロパティを追加します
        * PHPのオブジェクトは動的にプロパティを追加することができます。好きなプロパティ名を使用することもできます
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // クライアントの実際のIPアドレスを使用する場合、$connection->realIPを直接使用します
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**GatewayWorkerがNginxの設定からヘッダーを取得する方法**

Events.phpに以下のコードを追加します
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... 他のコードは省略....
}
```
コードを追加したら、GatewayWorkerを再起動する必要があります。

これで、Events.phpの`onMessage`と`onClose`メソッドで`$_SESSION['realIP']`を使用してクライアントの実際のIPを取得できます。

> 注意：Events.phpの`onWorkerStart` `onConnect` `onWorkerStop`では`$_SESSION['realIP']`を直接使用することはできません。
