# text protokolü
> Workerman, metin adlı bir metin protokolü tanımlar, protokol formatı ```veri paketi + satır sonu``` olarak, yani her veri paketinin sonuna bir satır sonu eklenerek paketin sonu belirtilir.

Örneğin, aşağıdaki buffer1 ve buffer2 dizgeleri text protokolüne uygundur:

```php
// Metin ve bir satır sonu
$buffer1 = 'abcdefghijklmn
';
// PHP'de çift tırnak içindeki \n bir satır sonunu temsil eder, örneğin "\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// Sunucu ile bir soket bağlantısı oluştur
$client = stream_socket_client('tcp://127.0.0.1:5678');
// buffer1 verisini text protokolü ile gönder
fwrite($client, $buffer1);
// buffer2 verisini text protokolü ile gönder
fwrite($client, $buffer2);
```

text protokolü son derece basit ve kullanımı kolaydır. Geliştiricilerin kendi protokollerine ihtiyaç duymaları durumunda, örneğin, mobil uygulamalarla veri iletmek veya donanımla iletişim kurmak gibi durumlarda, text protokolünü kullanmayı düşünmeleri tavsiye edilir, geliştirme ve hata ayıklama son derece kolaydır.

**text protokolü hata ayıklama**

> text protokolü telnet istemcisi kullanılarak hata ayıklanabilir, örneğin aşağıdaki örnek:

Yeni bir test.php dosyası oluştur

```php
require_once __DIR__ . '/Workerman/Autoloader.php';
use Workerman\Worker;

$text_worker = new Worker("text://0.0.0.0:5678");

$text_worker->onMessage =  function($connection, $data)
{
    var_dump($data);
    $connection->send("hello world");
};

Worker::runAll();
```

```php test.php start``` komutunu çalıştırarak gösterilenleri görüntüleyin

```bash
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

Yeniden bir terminal açın ve telnet ile test edin (linux telnet'i kullanmanızı öneririm)

Varsayalım ki yerelde test ediyorsunuz,
Terminalden telnet 127.0.0.1 5678 komutunu çalıştırın
Ardından hi yazın ve enter tuşuna basın
"hello world\n" verilerini alacaksınız
```bash
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
