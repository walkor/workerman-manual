# Temel Akış
(Bir basit WebSocket sohbet odası sunucusu örneği olarak)

#### 1. Herhangi bir konumda proje dizini oluşturun
Örneğin SimpleChat/
Dizine girin ve `composer require workerman/workerman` komutunu çalıştırın.

#### 2. `vendor/autoload.php` dosyasını içe aktarın (Yükleyicinin oluşturduğu composer sonrası)
start.php dosyası oluşturun ve `vendor/autoload.php` dosyasını içe aktarın.
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. Protokol Seçimi
Burada Text metin protokolünü seçiyoruz (WorkerMan'de özel olarak metin+newline formatında bir protokol)

(Şu anda WorkerMan HTTP, WebSocket, Text metin protokolünü desteklemektedir. Diğer protokoller gerekiyorsa, kendi protokolünüzü geliştirmek için protokol bölümüne bakınız.)

#### 4. Gereksinimlere Göre Ana Başlangıç Betiğini Yazın
Aşağıda, basit bir sohbet odasının giriş dosyası bulunmaktadır.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// Bir istemci bağlandığında uid atayın ve bağlantıyı kaydedin ve tüm istemcilere bildirin
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // Bu bağlantıya bir uid atayın
    $connection->uid = ++$global_uid;
}

// İstemci mesaj gönderdiğinde, tüm kişilere iletin
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// İstemci bağlantısı koptuğunda, tüm istemcilere yayınlayın
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}

// 2347 portunu dinleyen bir metin protokolü Worker oluşturun
$text_worker = new Worker("text://0.0.0.0:2347");

// Yalnızca 1 işlem başlatın, bu şekilde istemciler arasındaki veri iletimi daha kolay olur
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();
```

#### 5. Test
Text protokolü telnet komutuyla test edilebilir
```shell
telnet 127.0.0.1 2347
```
