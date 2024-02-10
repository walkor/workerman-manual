# Açıklama

Workerman, 4.x sürümünden itibaren HTTP hizmetlerini desteklemeye başlamıştır. Bir istek sınıfı, yanıt sınıfı, oturum sınıfı ve [SSE](SSE.md) gibi yapılar tanıtmıştır. Workerman'in HTTP hizmetlerini kullanmayı düşünüyorsanız, Workerman 4.x veya daha yeni sürümlerini kesinlikle öneririz.

**Aşağıdaki kullanımların hepsi Workerman 4.x sürümüne aittir ve Workerman 3.x ile uyumlu değildir.**

# Oturum Nesnesini Almak
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Worker'ı başlat
Worker::runAll();
```
**Notlar**
- Oturum, `$connection->send()` çağrısından önce işlem görmelidir.
- Oturum nesnesi, obje yok edildiğinde otomatik olarak değişiklikleri kaydeder, bu nedenle `$request->session()` tarafından döndürülen nesneyi oturumun kaydedilememesine neden olacak şekilde global bir dizi veya sınıf üyesinde saklamamanız gerekir.
- Oturum varsayılan olarak disk dosyasına kaydedilir, daha iyi performans için redis kullanmanızı tavsiye ederiz.

## Tüm oturum verilerini almak
```php
$session = $request->session();
$all = $session->all();
```
Dizi şeklinde bir değer döner. Herhangi bir oturum verisi yoksa boş bir dizi döner.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send(var_export($session->all(), true));
};

// Worker'ı başlat
Worker::runAll();
```

## Belirli bir oturum değerini almak
```php
$session = $request->session();
$name = $session->get('name');
```
Veri mevcut değilse null olarak döner.

Ayrıca get yöntemine ikinci parametre olarak bir varsayılan değer iletebilirsiniz; oturum dizisinde karşılık gelen değer bulunamazsa varsayılan değeri döndürür. Örnek olarak:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// Worker'ı başlat
Worker::runAll();
```

## Oturumu saklamak
Bir veriyi saklamak için set yöntemi kullanılır.
```php
$session = $request->session();
$session->set('name', 'tom');
```
set yöntemi geriye bir değer döndürmez, oturum nesnesi yok edildiğinde oturum otomatik olarak kaydedilir.

Birden fazla değer saklarken put yöntemi kullanılır.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Benzer şekilde, put yöntemi de bir değer döndürmez.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send($session->get('name'));
};

// Worker'ı başlat
Worker::runAll();
```

## Oturum verisini silmek
Bir veya birden çok oturum verisini silmek için `forget` yöntemi kullanılır.
```php
$session = $request->session();
// Bir değeri sil
$session->forget('name');
// Birden çok değeri sil
$session->forget(['name', 'age']);
```

Ayrıca sistem, forget yönteminin yanı sıra sadece bir öğeyi silebilen delete yöntemini de sunar.
```php
$session = $request->session();
// $session->forget('name'); ile aynı
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// Worker'ı başlat
Worker::runAll();
```

## Oturumdan bir değer alıp silmek
```php
$session = $request->session();
$name = $session->pull('name');
```
Bu, aşağıdaki kodla aynı etkiye sahiptir:
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
Karşılık gelen oturum yoksa null döner.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->session()->pull('name'));
};

// Worker'ı başlat
Worker::runAll();
```

## Tüm oturum verilerini silmek
```php
$request->session()->flush();
```
Bir değer döndürmez, oturum nesnesi yok edildiğinde otomatik olarak depolamadan silinir.

**Örnek**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $request->session()->flush();
    $connection->send('ok');
};

// Worker'ı başlat
Worker::runAll();
```

## Belirli bir oturum verisinin varlığını kontrol etmek
```php
$session = $request->session();
$has = $session->has('name');
```
Yukarıdaki kod, ilgili oturum mevcut değilse veya oturum değeri null ise false döndürür, aksi halde true döner.


```php
$session = $request->session();
$has = $session->exists('name');
```
Yukarıdaki kod da oturum verisinin varlığını kontrol etmek için kullanılır, fark, ilgili oturum öğesinin değerinin null olması durumunda bile true döndürmesidir.


