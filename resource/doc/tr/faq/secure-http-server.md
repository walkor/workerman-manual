# SSL Sertifikası Oluşturma

**Soru:**

Workerman nasıl bir [https](https://baike.baidu.com/item/https) hizmeti oluşturabilir ve istemcilerin [https](https://baike.baidu.com/item/https) protokolünü kullanarak iletişim kurmasını sağlayabilir?

**Cevap:**

[https](https://baike.baidu.com/item/https) protokolü aslında [http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl)'dir, yani [http](https://baike.baidu.com/item/http) protokolüne [SSL](https://baike.baidu.com/item/ssl) katmanının eklenmesidir. Workerman [http](https://baike.baidu.com/item/http) protokolünü desteklerken aynı zamanda [SSL](https://baike.baidu.com/item/ssl)'yi de destekler (`Workerman versiyonu>=3.3.7 gerektirir`), bu nedenle [https](https://baike.baidu.com/item/https) protokolünü desteklemek için sadece [http](https://baike.baidu.com/item/http) protokolünün temelinde [SSL](https://baike.baidu.com/item/ssl)'yi etkinleştirmeniz yeterlidir.

workerman'ı https desteği için iki genel yöntem vardır. Birisi workerman'ın doğrudan SSL'yi etkinleştirmesi, diğeri ise SSL'yi nginx ile proxy olarak kullanmaktır. Her iki yöntemden birini seçin, aynı anda ayarlanmamalıdır.

## Workerman SSL'yi Etkinleştirme

**Hazırlık:**

1. Workerman versiyonu>=3.3.7
2. PHP'ye openssl eklentisi kurulmuş olmalıdır.
3. Sertifika (pem/crt dosyası ve key dosyası) `/etc/nginx/conf.d/ssl` altına yerleştirilmiş olmalıdır.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // crt dosyası da olabilir
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Kendi kendine imzalı bir sertifika ise bu seçeneği etkinleştirmeniz gerekebilir
    )
);

$worker = new Worker('http://0.0.0.0:443', $context);
$worker->transport = 'ssl'; // ssl'yi etkinleştirme
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Yukarıdaki kodla Workerman'da https hizmeti oluşturulur; istemciler artık güvenli şifrelemeyle iletişim kurmak için https protokolünü kullanabilir.

**Test:**

Tarayıcı adres çubuğuna `https://domainAdı:443` yazarak erişebilirsiniz.

**Notlar:**

1. https portuna sadece https protokolü ile erişilebilir, http protokolü ile erişilemez.
2. Sertifika genellikle bir alan adıyla ilişkilendirilmiştir, bu nedenle test yaparken bir alan adı kullanın, IP adresini kullanmayın.
3. https ile erişemiyorsanız sunucu güvenlik duvarını kontrol edin.

## SSL Proxy'si Olarak Nginx Kullanma

Workerman'ın kendi SSL'si yerine nginx'i bir SSL proxy'si olarak kullanarak da https'yi gerçekleştirebilirsiniz.

> **Not**
> nginx SSL proxy'si ve Workerman SSL ayarı birbirinden birini seçilmelidir, aynı anda etkinleştirilemez.

İletişim prensibi ve akışı şu şekildedir:

1. İstemci https bağlantısı isteği ile nginx'e bağlanır.
2. Nginx https protokolünü http protokolüne dönüştürür ve işlemek üzere Workerman'ın http portuna yönlendirir.
3. Workerman veriyi alır, iş mantığı işlemlerini gerçekleştirir ve http protokolünde veri döner.
4. Nginx http protokolünü https'e dönüştürür ve istemciye iletir.

### Nginx Yapılandırma Örneği
**Ön koşullar ve hazırlık:**

1. Varsayalım ki Workerman 8181 portunu dinliyor (http protokolü)
2. Sertifika (pem/crt dosyası ve key dosyası) `/etc/nginx/conf.d/ssl` altına yerleştirilmiş olmalıdır.
3. Nginx'in 443 portunu wss proxy servisi olarak açmayı planlıyorsunuz (gerektiğinde portu değiştirebilirsiniz).

**Nginx örneği:**

```nginx
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}

server {
  listen 443;
  server_name siteDomaini.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location / {
    proxy_pass http://workerman; // Workerman'a yönlendirme
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
