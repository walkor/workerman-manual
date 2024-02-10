# Açıklama
Workerman, 4.x sürümünden itibaren HTTP hizmetini güçlendirdi. Bir istek sınıfı, bir yanıt sınıfı, bir oturum sınıfı ve [SSE](SSE.md) entegre edildi. Workerman'in HTTP hizmetlerini kullanmak istiyorsanız, workerman 4.x veya daha yeni sürümlerini şiddetle öneririz.

**Aşağıdaki tüm kullanımlar workerman 4.x sürümleri içindir ve workerman 3.x sürümleriyle uyumlu değildir.**

## İstek Objesine Erişim
İstek objesi, her durumda onMessage geri çağrı işlevinde alınır. Çerçeve otomatik olarak Request objesini geri çağırma işlevinin ikinci parametresi olarak iletecektir.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request istek objesi, burada herhangi bir işlem yapmadan doğrudan tarayıcıya hello mesajını gönderiyor.
    $connection->send("hello");
};

// İşçiyi çalıştırma
Worker::runAll();
```

Tarayıcı `http://127.0.0.1:8080` adresine eriştiğinde `hello` yanıtını alacaktır.

## GET İstek Parametrelerini Almak

**Tüm GET dizisini almak**
```php
$get = $request->get();
```
Eğer istekte GET parametresi yoksa boş bir dizi döndürülür.

**Belirli bir GET değerini almak**
```php
$name = $request->get('name');
```
Eğer GET dizisi bu değeri içermiyorsa null döner.

Ayrıca get yöntemine ikinci bir parametre olarak bir varsayılan değer iletebilirsiniz. Eğer GET dizisi içinde ilgili değer bulunamazsa varsayılan değer dönecektir. Örneğin:
```php
$name = $request->get('name', 'tom');
```

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->get('name'));
};

// İşçiyi çalıştırma
Worker::runAll();
```

Tarayıcı `http://127.0.0.1:8080?name=jerry&age=12` adresine eriştiğinde `jerry` yanıtını alacaktır.

## POST İstek Parametrelerini Almak
**Tüm POST dizisini almak**
```php
$post = $request->post();
```
Eğer istekte POST parametresi yoksa boş bir dizi döndürülür.

**Belirli bir POST değerini almak**
```php
$name = $request->post('name');
```
Eğer POST dizisi bu değeri içermiyorsa null döner.

GET yöntemiyle aynı şekilde, post yöntemine ikinci bir parametre olarak bir varsayılan değer iletebilirsiniz. Eğer POST dizisi içinde ilgili değer bulunamazsa varsayılan değer dönecektir. Örneğin:
```php
$name = $request->post('name', 'tom');
```
**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = $request->post();
    $connection->send(var_export($post, true));
};

// İşçiyi çalıştırma
Worker::runAll();
```


## Raw Body İle İstek Post Paketini Almak
```php
$post = $request->rawBody();
```
Bu özellik, `php-fpm` içindeki `file_get_contents("php://input");` işlemine benzer. HTTP özgün istek paketini almak için kullanılır. Bu, `application/x-www-form-urlencoded` formatında olmayan post istek verilerini almak için çok kullanışlıdır.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = json_decode($request->rawBody());
    $connection->send('hello');
};

// İşçiyi çalıştırma
Worker::runAll();
```

## Header Almak
**Tüm header dizisini almak**
```php
$headers = $request->header();
```
Eğer istekte header parametresi yoksa boş bir dizi döndürülür. Tüm anahtarlar küçük harfle döndürülür.

**Belirli bir header değerini almak**
```php
$host = $request->header('host');
```
Eğer header dizisi bu değeri içermiyorsa null döner. Tüm anahtarlar küçük harfle döndürülür.

GET yöntemiyle aynı şekilde, header yöntemine ikinci bir parametre olarak bir varsayılan değer iletebilirsiniz. Eğer header dizisi içinde ilgili değer bulunamazsa varsayılan değer dönecektir. Örneğin:
```php
$host = $request->header('host', 'localhost');
```
**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->header('connection') === 'keep-alive') {
        $connection->send('hello');
    } else {
        $connection->close('hello');
    }    
};

// İşçiyi çalıştırma
Worker::runAll();
```

## Cookie Almak
**Tüm cookie dizisini almak**
```php
$cookies = $request->cookie();
```
Eğer istekte cookie parametresi yoksa boş bir dizi döndürülür.

**Belirli bir cookie değerini almak**
```php
$name = $request->cookie('name');
```
Eğer cookie dizisi bu değeri içermiyorsa null döner.


GET yöntemiyle aynı şekilde, cookie yöntemine ikinci bir parametre olarak bir varsayılan değer iletebilirsiniz. Eğer cookie dizisi içinde ilgili değer bulunamazsa varsayılan değer dönecektir. Örneğin:
```php
$name = $request->cookie('name', 'tom');
```
**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $cookie = $request->cookie();
    $connection->send(var_export($cookie, true));
};

// İşçiyi çalıştırma
Worker::runAll();
```

## Dosya Yüklemeleri Almak
**Tüm dosya yüklemeleri dizisini almak**
```php
$files = $request->file();
```
Dönen dosya formatı şuna benzer:
```php
array (
    'avatar' => array (
            'name' => '123.jpg',
            'tmp_name' => '/tmp/workerman.upload.9hjR4w',
            'size' => 1196127,
            'error' => 0,
            'type' => 'application/octet-stream',
      ),
     'anotherfile' =>  array (
            'name' => '456.txt',
            'tmp_name' => '/tmp/workerman.upload.9sirSws',
            'size' => 490,
            'error' => 0,
            'type' => 'text/plain',
      )
)
```
Burada:
 
 - name dosya adıdır
 - tmp_name diskteki geçici dosya konumu
 - size dosya boyutu
 - error [hata kodu](https://www.php.net/manual/zh/features.file-upload.errors.php)
 - type dosya tipi.

**Notlar:**

 - Dosya yükleme boyutu [defaultMaxPackageSize](../tcp-connection/default-max-package-size.md) sınırına tabidir, varsayılan olarak 10M, değiştirilebilir.
 - İstek sona erdikten sonra dosyalar otomatik olarak temizlenecektir.
 - Eğer istekte dosya yüklemesi yoksa boş bir dizi döndürülür.

### Belirli bir dosya yüklemesi almak
```php
$avatar_file = $request->file('avatar');
```
Şuna benzer bir çıktı döner:
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
Eğer yüklenen dosya yoksa null döner.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = $request->file('avatar');
    if ($file && $file['error'] === UPLOAD_ERR_OK) {
        rename($file['tmp_name'], '/home/www/web/public/123.jpg');
        $connection->send('ok');
        return;
    }
    $connection->send('upload fail');
};

// İşçiyi çalıştırma
Worker::runAll();
```
## Host Almak
İstek yapılan host bilgisini alır.
```php
$host = $request->host();
```
Eğer isteğin adresi standart olmayan 80 veya 443 port değilse, host bilgisi portu ile birlikte gelebilir, örneğin `example.com:8080`. Port gerekmiyorsa, ilk parametre olarak `true` geçirilebilir.
```php
$host = $request->host(true);
```
**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->host());
};

// Worker'ı çalıştır
Worker::runAll();
```
Tarayıcı `http://127.0.0.1:8080?name=tom` adresine gittiğinde `127.0.0.1:8080` döndürür.

## İstek Yöntemini Almak
```php
$method = $request->method();
```
Dönüş değeri `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD` içinden biri olabilir.
**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->method());
};

// Worker'ı çalıştır
Worker::runAll();
```
## İstek URI'sini Almak
```php
$uri = $request->uri();
```
İstek URI'sini, path ve queryString kısmını içeren şekilde döndürür.
**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->uri());
};

// Worker'ı çalıştır
Worker::runAll();
```
Tarayıcı `http://127.0.0.1:8080/user/get.php?uid=10&type=2` adresine gittiğinde `/user/get.php?uid=10&type=2` döndürür.

## İstek Yolunu Almak
```php
$path = $request->path();
```
İstek path kısmını döndürür.
**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->path());
};

// Worker'ı çalıştır
Worker::runAll();
```
Tarayıcı `http://127.0.0.1:8080/user/get.php?uid=10&type=2` adresine gittiğinde `/user/get.php` döndürür.

## İstek QueryString'ini Almak
```php
$query_string = $request->queryString();
```
İstek queryString kısmını döndürür.
**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->queryString());
};

// Worker'ı çalıştır
Worker::runAll();
```
Tarayıcı `http://127.0.0.1:8080/user/get.php?uid=10&type=2` adresine gittiğinde `uid=10&type=2` döndürür.

## İstek HTTP Sürümünü Almak
```php
$version = $request->protocolVersion();
```
Döndürülen değer `1.1` veya `1.0` olabilir.
**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->protocolVersion());
};

// Worker'ı çalıştır
Worker::runAll();
```

## İstek Session ID'sini Almak
```php
$sid = $request->sessionId();
```
Harf ve rakamlardan oluşan bir dize döndürür.
**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->sessionId());
};

// Worker'ı çalıştır
Worker::runAll();
```
