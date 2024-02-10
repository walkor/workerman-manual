# Açıklama

Workerman, 4.x sürümünden itibaren HTTP hizmetlerini güçlendirmiştir. Bir istek sınıfı, yanıt sınıfı, oturum sınıfı ve [SSE](SSE.md) gibi özellikler eklenmiştir. Eğer Workerman'in HTTP hizmetlerini kullanmak istiyorsanız, Workerman 4.x veya daha yeni bir sürümünü kullanmanızı şiddetle öneririz.

**Aşağıdaki tüm kullanımlar Workerman 4.x sürümüyle uyumludur ve Workerman 3.x sürümüyle uyumlu değildir.**


# Önemli

- Chunk veya SSE yanıtı gönderilmediği sürece, bir istekte birden fazla yanıt gönderilmesine izin verilmez, yani bir istekte `$connection->send()` yöntemi birden fazla kez çağrılamaz.
- Her istekte sonunda `$connection->send()` yöntemi mutlaka çağrılmalıdır, aksi takdirde istemci sürekli bekleyecektir.

## Hızlı Yanıt
HTTP durum kodunu (varsayılan olarak 200), başlık veya çerez özelleştirmeniz gerekmediğinde, doğrudan bir dize göndererek yanıtı tamamlayabilirsiniz.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Doğrudan "Bu bir içerik" dizesini istemciye gönder
    $connection->send("Bu bir içerik");
};

// İşçiyi çalıştır
Worker::runAll();
``` 

## Durum Kodunu Değiştirme
Özel durum kodları, başlık veya çerez gerektiğinde, `Workerman\Protocols\Http\Response` yanıt sınıfını kullanmanız gerekir. Örneğin, aşağıdaki örnekte, `/404` yolu ziyaret edildiğinde 404 durum kodu ve içerik olarak `<h1>Özür dileriz, dosya bulunamadı</h1>` döndürülüyor.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->path() === '/404') {
        $connection->send(new Response(404, [], '<h1>Özür dileriz, dosya bulunamadı</h1>'));
    } else {
        $connection->send('Bu bir içerik');
    }
};

// İşçiyi çalıştır
Worker::runAll();
```

`Response` sınıfı bir kez başlatıldıktan sonra, durum kodunu aşağıdaki şekilde değiştirmek mümkündür.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## Başlık Gönderme
Benzer şekilde, başlık göndermek için `Workerman\Protocols\Http\Response` yanıt sınıfını kullanmanız gerekir.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [
        'Content-Type' => 'text/html',
        'X-Header-One' => 'Başlık Değeri'
    ], 'Bu bir içerik');
    $connection->send($response);
};

// İşçiyi çalıştır
Worker::runAll();
```

`Response` sınıfı bir kez başlatıldıktan sonra, başlık eklemek veya değiştirmek için aşağıdaki yöntemleri kullanabilirsiniz.
```php
$response = new Response(200);
// Bir başlık eklemek veya değiştirmek
$response->header('Content-Type', 'text/html');
// Birden fazla başlık eklemek veya değiştirmek
$response->withHeaders([
    'Content-Type' => 'application/ json',
    'X-Header-One' => 'Başlık Değeri'
]);
$connection->send($response);
```

## Yönlendirme
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)

$worker = new Worker('http://0.0.0.0:8080');
$worker->onMessage = function($connection, $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};
Worker::runAll();
```

## Çerez Gönderme
Benzer şekilde, çerez göndermek için `Workerman\Protocols\Http\Response` yanıt sınıfını kullanmanız gerekir.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [], 'Bu bir içerik');
    $response->cookie('name', 'tom');
    $connection->send($response);
};

// İşçiyi çalıştır
Worker::runAll();
```

## Dosya Gönderme
Benzer şekilde, dosya göndermek için `Workerman\Protocols\Http\Response` yanıt sınıfını kullanmanız gerekir.

Dosya gönderirken aşağıdaki yöntemi kullanabilirsiniz
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- Workerman büyük dosyaları göndermeyi destekler
- Büyük dosyalar (2MB'den fazla) için, Workerman tüm dosyayı belleğe birden fazla kez yüklemeyecek, uygun zamanlarda dosyayı parçalayacak ve gönderecektir
- Workerman, istemci tarafından alınan hızı dikkate alarak dosya okuma ve gönderme hızını optimize eder, en hızlı dosya gönderimini sağlarken bellek kullanımını en düşük seviyede tutar
- Veri gönderimi engellenmez, diğer istekleri etkimez
- Dosya gönderilirken, gelecekteki isteklerde sunucunun, dosyanın değiştirilip değiştirilmediğini anlaması ve dosya transferini optimize etmek için 304 yanıtını göndermesi için otomatik olarak `Last-Modified` başlığını ekler
- Gönderilen dosya, uygun `Content-Type` başlığını kullanarak tarayıcıya otomatik olarak gönderilir
- Eğer dosya mevcut değilse, otomatik olarak 404 yanıtına dönüşür

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = '/dosya/yolunuz';
    // Dosyanın değiştirilip değiştirilmediğini değerlendirmek için if-modified-since başlığını kontrol et
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // Dosya değiştirilmediyse 304 yanıtı gönder
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // Dosya değiştirildiyse veya if-modified-since başlığı yoksa dosyayı gönder
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// İşçiyi çalıştır
Worker::runAll();
```

## HTTP Chunk Verisi Gönderme
 - Müşteriye önce `Transfer-Encoding: chunked` başlıklı bir Yanıt yanıtı göndermek zorunludur
 - Sonraki chunk verileri için `Workerman\Protocols\Http\Chunk` sınıfı kullanılır
 - Sonunda, yanıtı bitirmek için boş bir chunk verilmesi gerekir

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Protocols\Http\Chunk;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // İlk olarak, `Transfer-Encoding: chunked` başlıklı bir Yanıt yanıtı gönder
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // Sonraki Chunk verileri için Workerman\Protocols\Http\Chunk sınıfı kullanılır
    $connection->send(new Chunk('İlk veri parçası'));
    $connection->send(new Chunk('İkinci veri parçası'));
    $connection->send(new Chunk('Üçüncü veri parçası'));
   // Sonunda, yanıtı bitirmek için boş bir chunk verilir
    $connection->send(new Chunk(''));
};

// İşçiyi çalıştır
Worker::runAll();
``` 
