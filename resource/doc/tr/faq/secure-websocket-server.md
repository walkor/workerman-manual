# wss hizmeti oluşturma

**Soru:**

Workerman nasıl bir wss hizmeti oluşturur, böylece istemciler wss protokolünü kullanarak iletişim kurabilir, örneğin, WeChat mini uygulamasının sunucuya bağlanması.

**Cevap:**

wss protokolü aslında [websocket](https://baike.baidu.com/item/WebSocket)+[SSL](https://baike.baidu.com/item/ssl)'dir, yani websocket protokolüne [SSL](https://baike.baidu.com/item/ssl) katmanını eklemektir, benzer şekilde [https](https://baike.baidu.com/item/https)([http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl)) gibi.
Bu nedenle, wss protokolünü desteklemek için websocket protokolüne [SSL](https://baike.baidu.com/item/ssl) açmanız yeterlidir. 

## Yöntem 1, nginx/apache ile SSL proxy (tavsiye edilir)

**İletişim prensibi ve akışı**

1. İstemci wss bağlantısını nginx/apache üzerinden başlatır.
2. Nginx/apache, wss protokolündeki verileri ws protokol verilerine dönüştürerek ve Workerman'ın websocket protokol portuna ileterek verileri iletecektir.
3. Workerman, verileri aldıktan sonra iş mantığı işlemi yapacaktır.
4. Workerman, istemciye mesaj gönderdiğinde, verileri nginx/apache üzerinden wss protokolüne dönüştürerek ve ardından istemciye iletecektir.

## nginx konfigürasyon örneği

**Ön koşullar ve hazırlık**

1. Nginx'in yüklenmiş olduğunu, sürümünün 1.3'ten düşük olmadığını kabul edelim.
2. Workerman'ın 8282 portunu dinleyen websocket protokolü olduğunu varsayalım.
3. Sertifika (pem/crt dosyası ve anahtar dosyası) zaten /etc/nginx/conf.d/ssl dizinine yerleştirildi.
4. 443 portunu wss proxy hizmeti sunmak için kullanmaya karar verildi.
5. Nginx genellikle web sunucusu olarak çalıştığı için, mevcut sitesi kullanımını etkilememesi için wss için giriş olarak ```domain.com/wss``` adresi kullanılmıştır. Yani istemci bağlantı adresi wss://domain.com/wss olacaktır.

**nginx konfigürasyonu aşağıdaki gibi olacaktır**:
```nginx
server {
  listen 443;
  # Domain configuration is omitted...

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
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} Diğer site yapılandırmaları...
}
```
**Test**
```javascript
// Sertifika, domain adını kontrol eder, lütfen domain adını kullanın. Burada bağlantı noktası belirtilmemiştir.
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Bağlantı başarılı");
    ws.send('tom');
    alert("Sunucuya bir dize gönderiliyor: tom");
};
ws.onmessage = function(e) {
    alert("Sunucudan gelen mesaj: " + e.data);
};
```

## apache ile wss proxy kullanma

Ayrıca, apache'yi wss proxy olarak kullanarak workerman'a yönlendirebilirsiniz.

Hazırlık:

1. GatewayWorker 8282 portunu dinleyen(websocket protokol) olduğunu varsayalım.
2. SSL sertifikası zaten /server/httpd/cert/ dizininde yerleştirildi.
3. 443 portunu belirli port 8282'ye yönlendirmek için apache kullanılmak istenmiştir.
4. httpd-ssl.conf dosyası yüklenmiş.
5. openssl yüklü.

**proxy_wstunnel_module modülünü etkinleştirme**
```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**SSL ve proxy yapılandırması**
```apache
#extra/httpd-ssl.conf
DocumentRoot "/website/directory"
ServerName domain

# Proxy Config
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# SSL protokol desteği ekleme, güvensiz protokolleri kapatma
SSLProtocol all -SSLv2 -SSLv3
# Şifreleme süitini aşağıdakine göre değiştir
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# Sertifika genel anahtar yapılandırması
SSLCertificateFile /server/httpd/cert/your.pem
# Sertifika anahtar yapılandırması
SSLCertificateKeyFile /server/httpd/cert/your.key
# Sertifika zinciri yapılandırması
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**Test**
```javascript
// Sertifika, domain adını kontrol eder, lütfen domain adını kullanın. Burada bağlantı noktası belirtilmemiştir.
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Bağlantı başarılı");
    ws.send('tom');
    alert("Sunucuya bir dize gönderiliyor: tom");
};
ws.onmessage = function(e) {
    alert("Sunucudan gelen mesaj: " + e.data);
};
```


## Yöntem 2, Workerman'ı doğrudan SSL ile başlatma (tavsiye edilmez)

> **Dikkat**
> Nginx/apache SSL proxy ve Workerman SSL yapılandırması iki seçenekten birini kullanılmalıdır, aynı anda ikisi de açık olamaz.

**Hazırlık:**

1. Workerman sürümü >=3.3.7
2. PHP'nin openssl uzantısı yüklü
3. Sertifika (pem/crt dosyası ve anahtar dosyası) herhangi bir disk dizinine yerleştirildi

**Kod:**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// En iyi seçenek sertifika alınmış sertifikadır
$context = array(
    // Daha fazla ssl seçeneği için kılavuza bakın http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // Mutlak yol kullanın
        'local_cert'        => 'disk_path/server.pem', // ya da crt dosyası da olabilir
        'local_pk'          => 'disk_path/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Kendi imzalı bir sertifika kullanılacaksa bu seçeneği açmanız gerekir.
    )
);
// Burada websocket protokolü ayarlandı (port herhangi seçilebilir, ancak başka bir program tarafından kullanılmadığından emin olunmalıdır)
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// transport SSL'i başlatır, websocket+SSL yani wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Yukarıdaki kod sayesinde, Workerman wss protokolünü dinler ve istemciler güvenli anlık iletişim için wss protokolü kullanarak Workerman'a bağlanabilir.

**Test**

Chrome tarayıcısını açın, F12 tuşuna basarak hata ayıklama konsolunu açın, Konsol sekmesine aşağıdaki kodu girin (veya aşağıdaki kodu html sayfasına yerleştirerek çalıştırın)

```javascript
// Sertifika, domain adını kontrol eder, lütfen domain adını kullanın, burada bağlantı noktası var.
ws = new WebSocket("wss://domain.com:8282");
ws.onopen = function() {
    alert("Bağlantı başarılı");
    ws.send('tom');
    alert("Sunucuya bir dize gönderiliyor: tom");
};
ws.onmessage = function(e) {
    alert("Sunucudan gelen mesaj: " + e.data);
};
```

**Not:**

1. 443 portunu kullanmanız gerekiyorsa, lütfen ilk önerilen yöntemi kullanın, nginx/apache proxy yöntemini kullanarak wss'i etkinleştirin.
2. wss portu yalnızca wss protokolüyle erişilebilir, ws wss portuna erişemez.
3. Sertifika genellikle bir etki alanıyla ilişkilendirildiği için test sırasında lütfen istemci tarafı için etki alanı kullanın, ip adresini kullanmayın.
4. Erişilemezlik durumunda, lütfen sunucu güvenlik duvarını kontrol edin.
5. Bu yöntem, PHP sürümü> = 5.6 gerektirir, çünkü WeChat mini programları tls1.2 gerektirir ve PHP5.6'dan önceki sürümler tls1.2'yi desteklemez.
