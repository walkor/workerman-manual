# Şifreleme - ssl/tls

**Soru:**

Workerman ile iletişimin güvenliğini nasıl sağlanır?

**Cevap:**

İletişim protokolünün üzerine [SSL](https://baike.baidu.com/item/ssl) şifreleme katmanı ekleyerek (örneğin wss, [https](https://baike.baidu.com/item/https) protokolleri gibi) iletişimi güvenli hale getirmenin en kolay yolu budur. Workerman, [SSL](https://baike.baidu.com/item/ssl) desteği (```Workerman versiyon 3.3.7 veya daha yeni bir sürüm gerektirir```) sunar ve SSL'yi etkinleştirmek için yalnızca bazı ayarlamalar yapmanız gerekir.

Tabii ki, geliştiriciler kendi şifreleme/çözme algoritmaları üzerine bir mekanizma oluşturabilirler.

## Workerman'da SSL'nin etkinleştirilmesi aşağıdaki gibidir:


**Hazırlık İşlemleri:**

1. Workerman sürümü 3.3.7'den büyük olmalıdır.

2. PHP'de openssl eklentisi yüklü olmalıdır.

3. Sertifika (pem/crt dosyası ve key dosyası) zaten /etc/nginx/conf.d/ssl altına yerleştirilmiş olmalıdır.

**Kod:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// En iyi şekilde bir sertifika alınmış olmalı
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // veya crt dosyası olabilir
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Kendi imzalı bir sertifika ise bu seçeneği etkinleştirmeniz gerekir
    )
);
// Burada websocket protokolü ayarlanmıştır. Aynı şekilde http protokolü veya diğer protokoller de kullanılabilir
$worker = new Worker('websocket://0.0.0.0:443', $context);
// Transport SSL olarak ayarlanır
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Workerman'da Sunucu Adı Gösterimi [SNI (Server Name Indication)](https://baike.baidu.com/item/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA) etkinleştirme

Aynı IP ve port durumunda, çoklu sertifika bağlamak için kullanılır.

**.pem ve .key dosyalarını birleştirme:**

Her sertifikaya ait .pem ve ilgili .key dosyalarının içeriğini birleştirmeli ve .key dosyasının içeriğini .pem dosyasının sonuna eklemelisiniz. (Eğer .pem dosyası özel anahtarı içeriyorsa bu adımı atlayabilirsiniz.)

**Lütfen dikkat, tek bir sertifika, tüm sertifikaların aynı dosyaya kopyalanması anlamına gelmez**

Örneğin *host1.com.pem* dosyasının birleştirilmiş .pem dosyasının içeriği aşağıdaki gibi olabilir:

```text
-----BEGIN CERTIFICATE-----
MIIGXTCBA...
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIFBzCCA...
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAA....
-----END RSA PRIVATE KEY-----
```

**Kod:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // SNI etkinleştirme
        'SNI_server_certs' => [ // Çoklu sertifika ayarları
            'host1.com' => '/path/host1.com.pem', // Sertifika 1
            'host2.com' => '/path/host2.com.pem', // Sertifika 2
        ],
        'local_cert' => '/path/default.com.pem', // Varsayılan sertifika
        'local_pk'   => '/path/default.com.key',
    )
);
$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
