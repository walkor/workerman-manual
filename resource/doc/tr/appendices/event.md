```php
# webman'ın şu anda desteklediği olay iticiliği

| Ad  | Bağlı Uzantı | Koşullu Destek |  Öncelik  |  webman Sürüm  |
|-----|------|--|-----|
|  Workerman\Events\Select   |   Yok   | Desteklenmiyor  |  Çekirdek Varsayılan Destek   |  >=3.0  |
|  Workerman\Events\Revolt   |   event(isteğe bağlı)   | Destekleniyor |  [revolt/event-loop](https://github.com/revoltphp/event-loop) kurulmalı   |  >=5.0  |
|  Workerman\Events\Event   |   event   | Desteklenmiyor |  Çekirdek Varsayılan Destek   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | Destekleniyor |  Elle ayarlanmalı   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | Destekleniyor |  Elle ayarlanmalı   |  >=5.0  |

* Her çekirdek iticiliği ayrı özellikler sunar, örneğin `Revolt` kullanıldığında workerman'ın PHP'nin dahili [Fiber korutini](https://www.php.net/manual/zh/language.fibers.php)'yi desteklemesini sağlar, `Swoole` kullanıldığında workerman'ın Swoole'un korutinisini desteklemesini sağlar
* Her olay iticiliği birbirleriyle karşıt ilişkilidir, örneğin `Revolt`'un Fiber korutini kullanımında, Swoole veya Swow'un korutinleri kullanılamaz
* `Revolt`'u kurmak için `composer require revolt/event-loop ^1.0.0` komutunu kullanmalısınız, kurulduktan sonra workerman çekirdeği otomatik olarak bunu tercih edilen olay iticiliği olarak ayarlar
* `Swoole` ve `Swow`'u etkinleştirmek için `Worker::$eventLoopClass` el ile ayarlanmalıdır (aşağıdaki paragrafa bakınız)
* Swoole varsayılan olarak [Kolay Korutin Çalışma Zamanı](https://wiki.swoole.com/#/runtime?id=runtime)'ı etkinleştirmemiştir, yani Pdo, Redis, PHP'nin dahili dosya okuma/yazması için hala engelleyici çağrılar olacaktır
* Swoole'un Kolay Korutin'u etkinleştirmesi gerekiyorsa, el ile `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` komutunu çağırmanız gerekir

> **Not**
> swow uzantısı bazı PHP'nin dahili işlevlerin davranışını otomatik olarak değiştirir, bu da swow'u etkinleştirdiğiniz halde swow'u olay iticiliği olarak kullanmadığınızda, workerman'ın isteklere ve sinyallere yanıt verememesine neden olur. Yani swow'u altta yatan sürücü olarak kullanmıyorsanız, swow'u php.ini'den yorumlamalısınız

Daha fazla ayrıntı için [workerman Coroutin](../fiber.md)'e bakın

# workerman için olay iticiliğini ayarlamak

Aşağıda workerman'ın olay iticiliğini el ile ayarlama örnekleri gösterilmiştir

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Altta yatan olay iticiliğini el ile ayarlamak
Worker::$eventLoopClass = Workerman\Events\Revolt::class;
//Worker::$eventLoopClass = Workerman\Events\Select::class;
//Worker::$eventLoopClass = Workerman\Events\Event::class;
//Worker::$eventLoopClass = Workerman\Events\Swoole::class;
//Worker::$eventLoopClass = Workerman\Events\Swow::class;
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
