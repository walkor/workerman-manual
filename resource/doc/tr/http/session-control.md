# Açıklama
Workerman, 4.x sürümünden itibaren HTTP hizmetlerini desteklemeye başlamıştır. Bir istek sınıfı, yanıt sınıfı, oturum sınıfı ve [SSE](SSE.md) gibi özellikler eklenmiştir. Eğer Workerman'ın HTTP hizmetlerini kullanmak istiyorsanız, Workerman 4.x veya daha yeni sürümlerini kesinlikle öneririz.

**Aşağıdaki kullanımların hepsi Workerman 4.x versiyonuna uygundur ve Workerman 3.x ile uyumlu değildir.**

## Oturum depolama motorunu değiştirme
Workerman, oturumlar için dosya depolama motoru ve redis depolama motoru sağlamaktadır. Varsayılan olarak dosya depolama motoru kullanılmaktadır. Redis depolama motorunu kullanmak istiyorsanız aşağıdaki kod örneğine bakınız.
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// redis yapılandırması
$config = [
    'host'     => '127.0.0.1', // Zorunlu parametre
    'port'     => 6379,        // Zorunlu parametre
    'timeout'  => 2,           // Opsiyonel parametre
    'auth'     => '******',    // Opsiyonel parametre
    'database' => 1,           // Opsiyonel parametre
    'prefix'   => 'session_'   // Opsiyonel parametre
];
// Workerman\Protocols\Http\Session::handlerClass yöntemini kullanarak oturumun temel sürücü sınıfını değiştirme
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## Oturum depolama konumunu ayarlama
Varsayılan depolama motorunu kullandığınızda, oturum verileri varsayılan olarak diske kaydedilir ve varsayılan konum `session_save_path()` fonksiyonunun dönüş değeridir. Depolama konumunu değiştirmek için aşağıdaki yöntemi kullanabilirsiniz.
```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Oturum dosyası depolama konumunu ayarlama
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Worker'ı çalıştırma
Worker::runAll();
```

## Oturum dosyalarının temizlenmesi
Varsayılan oturum depolama motorunu kullandığınızda, diskte birden çok oturum dosyası bulunabilir. Workerman, php.ini'de ayarlanan `session.gc_probability`, `session.gc_divisor` ve `session.gc_maxlifetime` seçeneklerine göre süresi dolmuş oturum dosyalarını temizler. Bu üç seçenek hakkında daha fazla bilgi için [php belgelerine](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability) bakınız.

## Depolama Sürücüsünü Değiştirme
Dosya oturum depolama motoru ve redis oturum depolama motoru dışında, Workerman standart [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) arabirimini kullanarak yeni oturum alanı depolama motoru eklemenize izin verir. Örneğin mangoDb oturum depolama motoru, MySQL oturum depolama motoru gibi.

**Yeni oturum depolama motoru ekleme süreci**
  1. [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) arabirimini uygulama
  2. `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` yöntemini kullanarak temel SessionHandler arabirimini değiştirme
 
**SessionHandlerInterface Arabirimi Uygulama**

Özel oturum depolama sürücüsü, SessionHandlerInterface arabirimini uygulamalıdır. Bu arabirim aşağıdaki yöntemleri içerir:
```php
SessionHandlerInterface {
    /* Yöntemler */
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**SessionHandlerInterface Açıklama**
 - read yöntemi, depolamadan session_id'ye karşılık gelen tüm session verilerini okumak için kullanılır. Verileri unserialize işlemi yapmayın, çünkü çerçeve bunu otomatik olarak yapacaktır.
 - write yöntemi, depolamaya session_id'ye karşılık gelen session verilerini yazmak için kullanılır. Verileri serialize işlemi yapmayın, çünkü çerçeve bunu otomatik olarak yapacaktır.
 - destroy yöntemi, session_id'ye karşılık gelen session verilerinin yok edilmesi için kullanılır.
 - gc yöntemi, süresi dolan session verilerini silmek için kullanılır, depolama maxlifetime'dan daha büyük olan tüm sessionları silmelidir
 - close, hiçbir işlem yapmasına gerek yok, sadece true değeri döndürmelidir
 - open, hiçbir işlem yapmasına gerek yok, sadece true değeri döndürmelidir

**Temel Sürücüyü Değiştirme**

SessionHandlerInterface arabirimini uyguladıktan sonra, oturumun temel sürücüsünü değiştirmek için aşağıdaki yöntemi kullanın.
```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name, SessionHandlerInterface arabirimini uygulayan SessionHandler sınıfının adıdır. Eğer bir namespace varsa tam namespace'i ile birlikte kullanılmalıdır
 - $config, SessionHandler sınıfının kurucu işlevi için parametrelerdir

**Spesifik İmplementasyon**

*Not: Bu MySessionHandler sınıfı sadece temel sürücüyü değiştirmenin adımlarını anlatmak içindir ve üretim ortamında kullanılamaz.*
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

class MySessionHandler implements SessionHandlerInterface
{
    protected static $store = [];
    
    public function __construct($config) {
        // ['host' => 'localhost']
        var_dump($config);
    }
   
    public function open($save_path, $name)
    {
        return true;
    }

    public function read($session_id)
    {
        return isset(static::$store[$session_id]) ? static::$store[$session_id]['content'] : '';
    }

    public function write($session_id, $session_data)
    {
        static::$store[$session_id] = ['content' => $session_data, 'timestamp' => time()];
    }

    public function close()
    {
        return true;
    }

    public function destroy($session_id)
    {
        unset(static::$store[$session_id]);
        return true;
    }

    public function gc($maxlifetime) {
        $time_now = time();
        foreach (static::$store as $session_id => $info) {
            if ($time_now - $info['timestamp'] > $maxlifetime) {
                unset(static::$store[$session_id]);
            }
        }
    }
}

//  Yeni uygulanan SessionHandler sınıfı bazı yapılandırmalara ihtiyaç duyuyorsa
$config = ['host' => 'localhost'];
//  Temel session sürücüsünü değiştirmek için Workerman\Protocols\Http\Session::handlerClass($class_name, $config) kullanılır
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
