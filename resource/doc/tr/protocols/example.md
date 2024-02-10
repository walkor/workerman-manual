# Bazı Örnekler

## Örnek Bir

### Protokol Tanımı
  * Tüm veri paketinin uzunluğunu saklamak için ilk 10 bayt sabit uzunluğa sahiptir, eksik haneler 0 ile doldurulur.
  * Veri formatı xml'dir.

### Veri Paketi Örneği
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
Burada 0000000121 tüm veri paketinin uzunluğunu temsil eder, ardından xml veri formatındaki paket içeriği takip eder.

### Protokol Uygulaması
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // 10 bayt yetersiz, veri beklemeye devam et
            return 0;
        }
        // Paket uzunluğunu döndür, paket uzunluğu başlık verisi uzunluğunu + paket vücut verisi uzunluğunu içerir
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // İstek paket vücudu
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        // Paket vücudu ve başlık uzunluğu
        $total_length = strlen($xml_string) + 10;
        // Uzunluğu 10 bayt yap, eksik haneler 0 ile doldur
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        // Veriyi döndür
        return $total_length_str . $xml_string;
    }
}
```

## Örnek İki

### Protokol Tanımı
  * İlk 4 bayt network byte order unsigned int, tüm paketin uzunluğunu belirtir.
  * Veri kısmı Json dizesidir.

### Veri Paketi Örneği
<pre>
****{"type":"message","content":"hello all"}
</pre>
Burada dört yıldızlı dört bayt network byte order unsigned int veriyi temsil eder, görünmeyen karakterlerdir, bunu takip eden Json veri formatındaki paket vücut verisidir.

### Protokol Uygulaması
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        // Gelen veri henüz 4 byteıfır yetmez, paket uzunluğu bilinemez, 0 döndür ve veri beklemeye devam et
        if(strlen($recv_buffer) < 4)
        {
            return 0;
        }
        // unpack fonksiyonu ile ilk 4 baytı sayıya dönüştür, ilk 4 byte tüm veri paketinin uzunluğudur
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        // İlk 4 byte'ı at, kalan kısmı paket Json verisi olarak al
        $body_json_str = substr($recv_buffer, 4);
        // Json decode
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        // Json encode ile paket vücudu oluştur
        $body_json_str = json_encode($data);
        // Tüm paketin uzunluğunu hesapla, ilk 4 byte + vücut verisinin byte sayısı
        $total_length = 4 + strlen($body_json_str);
        // Paketi pack et ve veri döndür
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## Örnek Üç (Binary protokolü ile dosya yükleme)

### Protokol Tanımı
```C
struct
{
  unsigned int total_len;  // Tüm paketin uzunluğu, büyük endian network byte order
  char         name_len;   // Dosya adının uzunluğu
  char         name[name_len]; // Dosya adı
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // Dosya verisi
}
```
### Protokol Örneği
<pre> *****logo.png****************** </pre>
Burada dört yıldızlı dört bayt network byte order unsigned int veriyi temsil eder, görünmeyen karakterlerdir, beşinci yıldız 1 byte ile dosya adının uzunluğunu saklar, ardından dosya adı takip eder, ardından orijinal ikili dosya verisi bulunur.

### Protokol Uygulaması
```php
namespace Protocols;
class BinaryTransfer
{
    // Protokol başlığı uzunluğu
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        // Eğer protokol başlığı uzunluğunu karşılayacak kadar veri yoksa, bekle
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        // Paketi unpack et
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // Paketin uzunluğunu döndür
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        // Paketi unpack et
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // Dosya adı uzunluğu
        $name_len = $package_data['name_len'];
        // Veri akışından dosya adını al
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        // Veri akışından ikili dosya verisini al
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // İstemciye gönderilecek veriyi kodlamak için kendi ihtiyaçlarınıza göre ayarlayabilirsiniz, burada sadece metin olarak dönerim
        return $data;
    }
}
```

### Sunucu Protokolü Kullanımı Örneği
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// Dosyayı tmp klasörüne kaydet
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("yükleme başarılı. kayıt yolu $save_path");
};

Worker::runAll();
```

### İstemci dosya client.php (Burada PHP ile istemci yükleme simüle edildi)
```php
<?php
/** Dosya yükleme istemcisi **/
// Yükleme adresi
$address = "127.0.0.1:8333";
// Yüklenecek dosya yol parametresini kontrol edin
if(!isset($argv[1]))
{
   exit("php client.php \$file_path şeklinde kullanın\n");
}
// Yüklenecek dosya yolu
$file_to_transfer = trim($argv[1]);
// Yerel dosya yok
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer yok\n");
}
// Soket bağlantısı oluştur
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// Engelli olarak ayarla
stream_set_blocking($client, 1);
// Dosya adı
$file_name = basename($file_to_transfer);
// Dosya adı uzunluğu
$name_len = strlen($file_name);
// Dosya ikili verisi
$file_data = file_get_contents($file_to_transfer);
// Protokol başlık uzunluğu 4 byte paket uzunluğu + 1 byte dosya adı uzunluğu
$PACKAGE_HEAD_LEN = 5;
// Protokol paketi
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// Yükleme işlemini gerçekleştir
fwrite($client, $package);
// Sonucu yazdır
echo fread($client, 8192),"\n";
```

### İstemci Kullanım Örneği
Komut satırında çalıştırın ```php client.php <dosya_yolu>```

Örneğin ```php client.php abc.jpg```
## Örnek Dört (Metin Protokolü Kullanarak Dosya Yükleme)

### Protokol Tanımı

json+newline, json içerisinde dosya adı ve base64_encode kodlu (boyutu 1/3 oranında artırır) dosya verisini içerir.

### Protokol Örneği

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

Not: Sonunda bir newline karakteri bulunmaktadır. PHP'de çift tırnak içinde ```"\n"``` karakteri ile belirtilir.

### Protokol Uygulaması
```php
namespace Protocols;
class TextTransfer
{
    public static function input($recv_buffer)
    {
        $recv_len = strlen($recv_buffer);
        if($recv_buffer[$recv_len-1] !== "\n")
        {
            return 0;
        }
        return strlen($recv_buffer);
    }

    public static function decode($recv_buffer)
    {
        // Paketi çöz
        $package_data = json_decode(trim($recv_buffer), true);
        // Dosya adını al
        $file_name = $package_data['file_name'];
        // base64_encode'dan alınan dosya verisini al
        $file_data = $package_data['file_data'];
        // base64_decode kullanarak orijinal ikili dosya verisine dönüştür
        $file_data = base64_decode($file_data);
        // Veriyi geri döndür
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // İstemciye gönderilecek veriyi kendi ihtiyacınıza göre kodlayabilirsiniz, burada sadece metin olarak geri dönmesini sağladık
        return $data;
    }
}
```

### Sunucu Protokolü Kullanımı Örneği
Açıklama: Binary yükleme yöntemi ile aynı şekilde yazılmıştır, yani neredeyse hiçbir iş mantığı kodunu değiştirmeden protokolü değiştirebilir.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// Dosyayı tmp klasörüne kaydet
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("Yükleme başarılı. Kayıt yolu $save_path");
};

Worker::runAll();
```
### İstemci Dosyası textclient.php (Burada bir PHP istemci yüklemesi simüle edilmiştir)
```php
<?php
/** Dosya yükleme istemcisi **/
// Yükleme adresi
$address = "127.0.0.1:8333";
// Yüklenecek dosya yolu parametresini kontrol et
if(!isset($argv[1]))
{
   exit("php client.php \$file_path şeklinde kullanın\n");
}
// Yüklenecek dosya yolu
$file_to_transfer = trim($argv[1]);
// Yüklenecek dosya yerelde mevcut değilse
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer dosyası mevcut değil\n");
}
// Soket bağlantısı oluştur
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// Dosya adı
$file_name = basename($file_to_transfer);
// Dosya ikili verisi
$file_data = file_get_contents($file_to_transfer);
// base64 kodlama
$file_data = base64_encode($file_data);
// Veri paketi
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// Protokol paketi json+newline
$package = json_encode($package_data)."\n";
// Yükleme işlemini gerçekleştir
fwrite($client, $package);
// Sonucu görüntüle
echo fread($client, 8192),"\n";
```

### İstemci Kullanımı Örneği
Komut istemine ```php textclient.php <dosya_yolu>``` şeklinde girin

Örneğin: ```php textclient.php abc.jpg```

