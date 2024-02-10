# Fiber (Elyaf) Koşutlu İşlemleri

Workerman, 5.0.0 sürümünden itibaren [Fiber (Elyaf) koşutlu işlemleri](https://www.php.net/manual/zh/language.fibers.php) desteklemektedir.

> **Not**
> Fiber özelliği için PHP>=8.1 ve `composer require revolt/event-loop ^1.0.0` kurulumu gerekmektedir.

### Tanıtım

Fiber, PHP'nin içinde bulunan bir koşutlu işlemdir ve PHP kodunu kesintiye uğratabilir, ardından ihtiyaç duyulduğunda çalışmasını devam ettirebilir. En büyük faydası, geliştiricilerin senkron bir şekilde asenkron ve engelsiz kod yazmalarına izin vererek kodun bakımı için büyük bir kolaylık sağlamasıdır.

### Örnek

Aşağıda, koşutlu işlem ve asenkron geri çağırma programlaması arasındaki farkı örneklerle karşılaştıralım.
Diyelim ki bir HTTP arabirimi çağrılması gerektiğini ve ardından cevap için bir saniye gecikme olması gerektiğini içeren bir gereksinimimiz var. Asenkron geri çağırma ve fiber kullanarak iki farklı yaklaşım aşağıdaki gibi olacaktır.

**Asenkron Geri Çağırma Yöntemi**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // HTTP arabirimini iste
    $http->get('http://example.com/', function ($response) use ($connection) {
        // Bir saniye geciktir
        Timer::add(1, function() use ($connection, $response) {
            // Tarayıcıya veri gönder
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**Fiber Kullanımı**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // HTTP arabirimini çağır
    $response = $http->get('http://example.com/');
    // Bir saniye gecikme
    Timer::sleep(1);
    // Veri gönder
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **Not**
> Yukarıdaki kodlar için `composer require workerman/http-client ^2.0.0` kurulumu gerekmektedir.

Bu iki yaklaşım da asenkron ve engelsiz bir şekilde çalışır; çalışma verimlilikleri oldukça iyidir, ancak fiber kullanımı asenkron geri çağırma yöntemine göre daha okunaklı ve bakımı daha kolaydır.

### Fiber Dikkat Edilmesi Gerekenler
* Fiber koşutlu işlemleri, Pdo, Redis ve PHP dahili engelleyici fonksiyonları koşutlu hale getirmeyi desteklemez, yani bu uzantıları ve fonksiyonları kullanmak hala engelleyici çağrılar olacaktır.
* Mevcut olan Fiber koşutlu işlem istemcileri [workerman/http-client](../components/workerman-http-client.md), [workerman/redis](../components/workerman-redis.md) adreslerinde bulunmaktadır.

# Swoole Koşutlu İşlemleri
Workerman v5, aynı anda Swoole'u altta yatan olay sürücüsü olarak desteklemektedir.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Burada Swoole'u altta yatan olay sürücü olarak ayarlamak gerekmektedir
Worker::$eventLoopClass = Workerman\Events\Swoole::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```
**İpucu**
* Swoole'ün 5.0 veya daha yeni bir sürümünü öneririz.
* Swoole'u altta yatan olay sürücü olarak kullanmak, workerman'ın Swoole'un koşutlu işlemlerini desteklemesini sağlar.
* Swoole'u altta yatan olay sürücü olarak kullanırken, event uzantısını kurmanıza gerek kalmaz.
* Swoole, varsayılan olarak tek tıklamalı koşutlu işlem özelliğini etkisizleştirmemiş şekilde gelir, yani Pdo, Redis, PHP dahili dosya okuma/yazma işlemleri engelleyici çağrılarla çalışır.
* Tek tıklamalı koşutlu işlem özelliğini etkinleştirmek için ` \Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` şeklinde manuel olarak çağrılması gerekmektedir.

Daha fazlası için [Swoole kılavuzu](https://wiki.swoole.com/) adresine göz atın.

Daha fazlası için [Olay Sürücüsü](appendices/event.md) bölümüne göz atın.

# Koşutlu İşlemler Hakkında
Öncelikle, koşutlu işleme aşırı derecede güvenmemekte fayda var. Veritabanı, Redis gibi depolamaları iç ağda olduğunda, çoklu süreç engelleyici çağrılar genellikle koşutlu işlemten daha hızlı olur. 
[techempower.com](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db) adresindeki 3 yıllık performans testi verilerine göre, workerman'ın engelleyici veritabanı çağırma performansı, swoole'un veritabanı havuzu ve koşutlu işlem kullanımından daha iyidir; hatta go dilinin gin, echo gibi koşutlu çerçevelerinden neredeyse 2 kat daha yüksek performans sunmaktadır.

Workerman, PHP uygulamasının performansını katlar veya onlarca kat artırır; çoğu workerman projesi, koşutlu işlemin ekstra performans kazandırmaktan çok daha fazla bir getirisi olmayacaktır.
Eğer sistemde yavaş çağrılar varsa, örneğin dış HTTP çağrıları gibi, performansı artırmak için koşutlu işlemi kullanmayı düşünebilirsiniz.
