```php
# Örnek 1
**``` (Workerman sürümü >=3.3.0 gerektirir) ```**

Çoklu işlemli Worker tabanlı grup ileti sistemine örnek

``` php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$channel_server = new Channel\Server('0.0.0.0', 2206);

$worker = new Worker('websocket://0.0.0.0:1234');
$worker->count = 8;
// Küresel grup bağlantı eşleştirme dizisi
$group_con_map = array();
$worker->onWorkerStart = function(){
    // Kanal istemciyi Kanal sunucusuna bağlan
    Channel\Client::connect('127.0.0.1', 2206);

    // Küresel grup gönderme olayını dinle
    Channel\Client::on('send_to_group', function($event_data){
        $group_id = $event_data['group_id'];
        $message = $event_data['message'];
        global $group_con_map;
        var_dump(array_keys($group_con_map));
        if (isset($group_con_map[$group_id])) {
            foreach ($group_con_map[$group_id] as $con) {
                $con->send($message);
            }
        }
    });
};
$worker->onMessage = function(TcpConnection $con, $data){
    // Gruba katılma  {"cmd":"add_group", "group_id":"123"} 
    // veya Gruba mesaj gönderme {"cmd":"send_to_group", "group_id":"123", "message":"Bu bir mesajdır"}
    $data = json_decode($data, true);
    var_dump($data);
    $cmd = $data['cmd'];
    $group_id = $data['group_id'];
    switch($cmd) {
        // Bağlantıyı gruba ekle
        case "add_group":
            global $group_con_map;
            // Bağlantıyı ilgili grup dizisine ekle
            $group_con_map[$group_id][$con->id] = $con;
            // Bu bağlantının hangi gruplara katıldığını kaydet, onclose sırasında group_con_map'e karşılık gelen grubun verilerini temizleme kolaylığı sağlar
            $con->group_id = isset($con->group_id) ? $con->group_id : array();
            $con->group_id[$group_id] = $group_id;
            break;
        // Gruba mesaj gönder
        case "send_to_group":
            // Kanal\Client tüm sunucuların tüm işlemlerine grup mesajı gönderme olayı
            Channel\Client::publish('send_to_group', array(
                'group_id'=>$group_id,
                'message'=>$data['message']
            ));
            break;
    }
};
// Burası çok önemli, bağlantı kapatıldığında bağlantıyı global grup verilerinden sil, bellek sızıntısını önlemek için
$worker->onClose = function(TcpConnection $con){
    global $group_con_map;
    // Bağlantının katıldığı tüm grupları döngüyle gezerek grup_con_map'ten karşılık gelen verileri sil
    if (isset($con->group_id)) {
        foreach ($con->group_id as $group_id) {
            unset($group_con_map[$group_id][$con->id]);
            if (empty($group_con_map[$group_id])) {
                unset($group_con_map[$group_id]);
            }
        }
    }
};

Worker::runAll();
```

## Test (Varsayalım ki her şey yerel makinede 127.0.0.1 üzerinde çalışıyor)

1. Sunucuyu başlatın
```bash
php start.php start
Workerman[del.php] DEBUG modunda çalıştırılıyor
----------------------- WORKERMAN -----------------------------
Workerman sürümü: 3.4.2          PHP sürümü: 7.1.3
------------------------ WORKERS -------------------------------
user          worker         listen                    processes durumu
liliang       ChannelServer  frame://0.0.0.0:2206       1         [OK] 
liliang       none           websocket://0.0.0.0:1234   12        [OK] 
----------------------------------------------------------------
Çıkmak için Ctrl-C tuşlarına basın. Başlatma başarılı.

```

2. İstemci, sunucuya bağlı

Tarayıcıyı açın ve F12 tuşuna basarak hata ayıklama konsolunu açın, Console'da aşağıdakine benzer bir şeyi (veya aşağıdaki kodları html sayfasına yerleştirerek js ile çalıştırabilirsiniz)

```javascript
// Sunucu ip'sini 127.0.0.1 olarak varsayalım, test ederken gerçek sunucu ip'sine değiştirin
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"Bu bir mesajdır"}');
};

```
