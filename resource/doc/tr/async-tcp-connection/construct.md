```php
# __construct Metodu
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
Bir asenkron bağlantı nesnesi oluşturur.

AsyncTcpConnection, Workerman'ın istemci olarak uzak sunucuya asenkron bağlantı başlatmasını ve send arayüzü ve onMessage geri çağrısını kullanarak bağlantıdaki verileri asenkron olarak göndermesini ve işlemesini sağlar.

## Parametreler
Parametre: ``` remote_address ```

Bağlantı adresi, örneğin
``` tcp://www.baidu.com:80 ```
``` ssl://www.baidu.com:443 ```
``` ws://echo.websocket.org:80 ```
``` frame://192.168.1.1:8080 ```
``` text://192.168.1.1:8080 ```

Parametre: ``` $context_option ```

```Bu parametre gereklidir (workerman >= 3.3.5)```

Socket bağlamını ayarlamak için kullanılır, örneğin```bindto``` ile dış ağa hangi (ağ kartı) IP ve port üzerinden erişim sağlanacağını, SSL sertifikası ayarlarını vb.

[stream_context_create](https://php.net/manual/en/function.stream-context-create.php), [Socket Context Options](https://php.net/manual/zh/context.socket.php), [SSL Context Options](https://php.net/manual/zh/context.ssl.php) referansına bakınız.

## Notlar

Şu anda AsyncTcpConnection tarafından desteklenen protokoller [tcp](https://baike.baidu.com/subview/32754/8048820.htm), [ssl](https://baike.baidu.com/view/525499.htm), [ws](appendices/about-ws.md), [frame](appendices/about-frame.md), [text](appendices/about-text.md) 'dir.

Ayrıca özel protokoller de desteklenir, bkz. [Nasıl Özel Protokol Oluşturulur](../protocols/how-protocols.md)

[ssl](https://baike.baidu.com/view/525499.htm) şu anda Workerman >= 3.3.4 gerektirir ve [openssl eklentisi](https://php.net/manual/zh/book.openssl.php) kurulu olmalıdır.

AsyncTcpConnection şu anda [http](https://baike.baidu.com/view/9472.htm) protokolünü desteklememektedir.

```new AsyncTcpConnection('ws://...')``` şeklinde Workerman'de websocket sunucusuna websocket isteği gönderebilirsiniz, bkz. [Örnekler](../appendices/about-ws.md). Ancak ```new AsyncTcpConnection('websocket://...')``` şeklinde websocket bağlantısı başlatılamaz.

## Örnekler

### Örnek 1: Dış HTTP hizmetine asenkron erişim
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// İşçi başladığında www.baidu.com'a asenkron olarak bir bağlantı oluşturur ve veri gönderir ve alır
$task->onWorkerStart = function($task)
{
    // Direkt olarak http belirtilmez ancak tcp kullanılarak http protokolü verileri gönderilebilir
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // Bağlantı başarıyla kurulduğunda, http isteği verilerini gönderir
    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "bağlantı başarılı\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "bağlantı kapandı\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Hata kodu:$code mesaj:$msg\n";
    };
    $connection_to_baidu->connect();
};

// İşçiyi çalıştır
Worker::runAll();
```

### Örnek 2: Dış WebSocket hizmetine asenkron erişim ve hangi yerel ip ve porttan erişileceğinin ayarlanması
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Karşıdaki ana bilgisayara hangi yerel ip ve porttan erişileceğinin ayarlanması (her soket bağlantısı bir yerel port kullanacaktır)
    $context_option = array(
        'socket' => array(
            // IP yerel ağ kartının IP'si olmalı ve karşıdaki ana bilgisayara erişilebilmelidir, aksi takdirde geçersiz
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('merhaba');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

### Örnek 3: Dış wss portuna asenkron erişim ve yerel ssl sertifikasının ayarlanması
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Karşıdaki ana bilgisayara hangi yerel ip ve porttan erişileceğinin ve ssl sertifikasının ayarlanması
    $context_option = array(
        'socket' => array(
            // IP yerel ağ kartının IP'si olmalı ve karşıdaki ana bilgisayara erişilebilmelidir, aksi takdirde geçersiz
            'bindto' => '114.215.84.87:2333',
        ),
        // ssl seçenekleri, ayrıntılar için bkz. https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Yerel sertifika yol. PEM formatında olmalıdır ve yerel sertifikayı ve özel anahtarı içermelidir.
            'local_cert'        => '/your/path/to/pemfile',
            // local_cert dosyasının şifresi
            'passphrase'        => 'your_pem_passphrase',
            // Kendi kendine imzalı sertifikanın kullanılabilmesi
            'allow_self_signed' => true,
            // SSL sertifikasının doğrulanıp doğrulanmayacağı
            'verify_peer'       => false
        )
    );

    // Asenkron bağlantı başlat
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // SSL şifreleme metodu olarak ayarlama
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('merhaba');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
