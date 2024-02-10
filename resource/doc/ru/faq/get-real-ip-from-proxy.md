Как получить реальный IP-адрес клиента через прокси nginx/apache?
Используя nginx/apache в качестве прокси для workerman, nginx/apache фактически выступает в роли клиента workerman, поэтому IP-адрес клиента, полученный в workerman, будет IP-адресом сервера nginx/apache, а не реальным IP-адресом клиента. Для получения реального IP-адреса клиента можно использовать следующий метод.

**Принцип:**

nginx/apache передает реальный IP-адрес клиента через заголовок HTTP, например, в конфигурации nginx внутри блока location добавьте ```proxy_set_header X-Real-IP $remote_addr;```. Workerman читает это значение из заголовка и сохраняет его в объекте $connection (GatewayWorker может сохранять в переменную $_SESSION), и при необходимости можно прочитать эту переменную.

**Примечание:**

Эти настройки подходят для протоколов http/https ws/wss. Для получения IP-адреса клиента для других протоколов аналогичным образом необходимо настроить прокси-сервер для передачи реального IP-адреса клиента.

**Пример настройки nginx:**
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
    # В этой части используется заголовок HTTP для передачи реального IP-адреса клиента
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} Другие настройки сайта...
}
```

**Workerman читает реальный IP-адрес клиента из заголовка, установленного nginx:**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// При подключении клиента, после завершения TCP-соединения
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * Обратный вызов при установке websocket-соединения с клиентом
    * В этом обратном вызове можно получить значение X_REAL_IP от nginx через заголовок HTTP
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * Объект connection изначально не имеет свойства realIP, здесь динамически добавляется свойство realIP в объект connection
        * Помните, что в PHP объекты могут динамически добавлять свойства, вы также можете использовать любое удобное для вас имя свойства
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // При использовании реального IP-адреса клиента $connection->realIP можно использовать напрямую
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**GatewayWorker получает IP-адрес клиента из установленного nginx заголовка:**
Добавьте следующий код в Events.php
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... Другой код ...
}
```
После добавления этого кода необходимо перезапустить GatewayWorker.

Таким образом, можно получить реальный IP-адрес клиента в методах `onMessage` и `onClose` Events.php через `$_SESSION['realIP']`.

> Примечание: В методах `onWorkerStart`, `onConnect`, `onWorkerStop` в Events.php нельзя напрямую использовать `$_SESSION['realIP']`.
