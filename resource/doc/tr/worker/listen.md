```php
void Worker::listen(void)
```
Worker örneği oluşturulduktan sonra dinleme işlemi için kullanılır.

Bu yöntem, Worker süreci başlatıldıktan sonra dinamik olarak yeni Worker örnekleri oluşturmak için kullanılır ve aynı süreçte birden fazla bağlantı noktasını dinleyebilir, çeşitli protokolleri destekler. Bu yöntemle sadece mevcut süreçte dinleme artar, yeni süreç oluşturmaz ve onWorkerStart yöntemini tetiklemez.

Örneğin, bir HTTP Worker başlatıldıktan sonra bir WebSocket Worker örneği oluşturursanız, bu süreç hem HTTP protokolüne hem de WebSocket protokolüne erişebilir. WebSocket Worker ve HTTP Worker aynı süreçte olduğu için ortak bellek değişkenlerine erişebilir, tüm soket bağlantılarını paylaşabilir. Bu, HTTP isteği almak ve ardından WebSocket istemcisini işleyerek istemciye veri gönderme gibi bir etkiyi gerçekleştirebilir.

**Not:**

Eğer PHP sürümü <= 7.0 ise, aynı portta Worker örneği oluşturmak için çoklu alt süreç desteklenmez. Örneğin A süreci 2016 portunu dinleyen bir Worker oluşturduğunda, B süreci 2016 portunu dinleyen bir Worker oluşturamaz, aksi takdirde "Address already in use" hatası alacaktır. Aşağıdaki kod **çalışmaz**:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

Worker::runAll();
```

Eğer PHP sürümünüz >=7.0 ise, Worker->reusePort=true olarak ayarlayabilirsiniz, böylece birden fazla alt süreç aynı portta Worker oluşturabilir. Aşağıdaki örneğe bakınız:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    $inner_worker->listen();
};
$worker->onMessage = 'on_message';
function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}
Worker::runAll();
```

### PHP Backend ile Client'a Anlık Mesaj Gönderme Örneği

**Prensip:**

1. Bir WebSocket Worker oluşturulur, müşteri ile uzun süreli bağlantıyı sürdürmek için
2. WebSocket Worker içinde bir text Worker oluşturulur
3. WebSocket Worker ve text Worker aynı süreçte olduğu için müşteri bağlantılarını kolayca paylaşabilir
4. Bağımsız bir PHP backend sistemi, text protokolü üzerinden text Worker ile haberleşir
5. text Worker, WebSocket bağlantısını kullanarak veri göndermeyi tamamlar

**Kod ve Adımlar:**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');

$worker->count = 1;
$worker->onWorkerStart = function($worker)
{
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        $ret = sendMessageByUid($uid, $buffer);
        $connection->send($ret ? 'ok' : 'fail');
    };
    $inner_text_worker->listen();
};
$worker->uidConnections = array();
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    if(!isset($connection->uid))
    {
       $connection->uid = $data;
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        unset($worker->uidConnections[$connection->uid]);
    }
};

function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

Worker::runAll();
```

Arka planda servisi başlat
 ```php push.php start -d```

Client tarafından alınan push.js kodu
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

Arka planda mesaj gönderen PHP kodu
```php
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
$data = array('uid'=>'uid1', 'percent'=>'88%');
fwrite($client, json_encode($data)."\n");
echo fread($client, 8192);
```
