# Geliştirme Standartları

## Uygulama Dizinleri

Uygulama dizinleri herhangi bir konuma yerleştirilebilir.

## Giriş Dosyası

Nginx+PHP-FPM'deki PHP uygulamaları gibi, Workerman'deki uygulamalar da bir giriş dosyasına ihtiyaç duyar. Giriş dosyasının bir adı zorunlu değildir ve bu giriş dosyası PHP Cli modunda çalıştırılır.

Giriş dosyası, dinleme işlemleriyle ilgili kodları içerir. Örneğin, aşağıdaki Worker tabanlı geliştirme kodu parçası:

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 2345 portunu dinleyen Worker oluştur, http iletişim protokolünü kullan
$http_worker = new Worker("http://0.0.0.0:2345");

// 4 adet işlemi başlatarak hizmet sun
$http_worker->count = 4;

// Tarayıcıdan gelen veriyi alıp tarayıcıya "hello world" cevabını gönder
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // "hello world" mesajını tarayıcıya gönder
    $connection->send('hello world');
};

Worker::runAll();

```

## Workerman'de Kod Standartları

1. Sınıflar büyük harfle başlayan CamelCase isimlendirmesi kullanır. Sınıf dosya adı, içerideki sınıf adı ile aynı olmalıdır ve otomatik yükleme için uygun olmalıdır. Örneğin:
```php
class UserInfo
{
...
```

2. Namespace kullanılmalıdır. Namespace adı, dizin yoluna karşılık gelmeli ve geliştiricinin proje kök dizinini baz almalıdır.

Örneğin, MyApp/ projesi için sınıf dosyası MyApp/MyClass.php ise, proje kök dizininde olduğundan namespace atanmaz. Fakat, MyApp/Protocols/MyProtocol.php dosyası MyApp projesinin Protocols dizininde olduğundan ```namespace Protocols;``` kullanılmalıdır.
```php
namespace Protocols;
class MyProtocol
{
....
```

3. Normal fonksiyonlar ve değişkenler küçük harf ve alt çizgiyle oluşturulmalıdır.
```php
$connection_list = array();
function get_connection_list()
{
....
```

4. Sınıf üyeleri ve sınıf metodları küçük harfle başlayan CamelCase şeklinde olmalıdır.
```php
public $connectionList;
public function getConnectionList();
```

5. Fonksiyon ve sınıf parametreleri küçük harf ve alt çizgiyle oluşturulmalıdır.
```php
function get_connection_list($one_param, $tow_param)
{
....

```
