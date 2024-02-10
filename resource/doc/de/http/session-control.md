# Erklärung
Ab der Version 4.x hat workerman die Unterstützung für HTTP-Dienste verbessert. Es wurden Request-Klassen, Response-Klassen, Session-Klassen und [SSE](SSE.md) eingeführt. Wenn Sie die HTTP-Dienste von workerman nutzen möchten, wird dringend empfohlen, workerman in der Version 4.x oder höher zu verwenden.

**Bitte beachten Sie, dass dies alles Verwendungsweisen für workerman 4.x-Versionen sind und nicht mit workerman 3.x kompatibel sind.**

## Ändern des Session-Speicher-Engines
workerman bietet ein Dateispeicher-Engine und ein Redis-Speicher-Engine für Sessions. Die Standardverwendung ist das Dateispeicher-Engine. Wenn Sie auf die Redis-Speicher-Engine umstellen möchten, folgen Sie bitte dem folgenden Code.
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// Redis-Konfiguration
$config = [
    'host'     => '127.0.0.1', // erforderlicher Parameter
    'port'     => 6379,        // erforderlicher Parameter
    'timeout'  => 2,           // optionale Parameter
    'auth'     => '******',    // optionale Parameter
    'database' => 1,           // optionale Parameter
    'prefix'   => 'session_'   // optionale Parameter
];
// Verwendung von Workerman\Protocols\Http\Session::handlerClass, um die Session-Treiberklasse zu ändern
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## Festlegen des Speicherorts für Sessions
Bei Verwendung des Standard-Speicher-Engine werden Sitzungsdaten standardmäßig auf der Festplatte gespeichert, mit dem Standardort, der vom Aufruf von `session_save_path()` zurückgegeben wird. Sie können den Speicherort mit den folgenden Methoden ändern.

```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Festlegen des Speicherorts für Sitzungsdateien
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Worker ausführen
Worker::runAll();
```

## Session-Dateireinigung
Bei Verwendung des Standard-Sitzungs-Speicher-Engine werden auf der Festplatte mehrere Sitzungsdateien erstellt. Workerman bereinigt veraltete Sitzungsdateien basierend auf den in der php.ini festgelegten Optionen `session.gc_probability`, `session.gc_divisor` und `session.gc_maxlifetime`. Weitere Informationen zu diesen drei Optionen finden Sie in der [php-Handbuch](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability)

## Ändern des Speicher-Treibers
Neben dem Datei-Sitzungs-Speicher-Engine und dem Redis-Sitzungs-Speicher-Engine können Sie mit workerman einen neuen Sitzungs-Speicher-Engine hinzufügen, indem Sie das Standard-[SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) verwenden, z.B. MangoDb-Sitzungs-Speicher-Engine, MySQL-Sitzungs-Speicher-Engine usw.

**Prozess zum Hinzufügen eines neuen Sitzungs-Speicher-Engine**
1. Implementieren Sie das [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) 
2. Verwenden Sie die Methode  `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)`, um den untergeordneten SessionHandler zu ersetzen

**Implementieren des SessionHandlerInterface**
Benutzerdefinierte Sitzungs-Speicher-Treiber müssen das SessionHandlerInterface implementieren, das folgende Methoden enthält:
```php
SessionHandlerInterface {
    /* Methoden */
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**Erklärung des SessionHandlerInterface**
- Die `read`-Methode wird verwendet, um alle Sitzungsdaten für die session_id aus dem Speicher zu lesen. Bitte führen Sie keine Deserialisierungsoperationen durch, da das Framework dies automatisch erledigt.
- Die `write`-Methode wird verwendet, um die Sitzungsdaten für die session_id in den Speicher zu schreiben. Bitte führen Sie keine Serialisierungsoperationen durch, da das Framework dies bereits automatisch erledigt.
- Die `destroy`-Methode wird verwendet, um die Sitzungsdaten für die session_id zu zerstören.
- Die `gc`-Methode wird verwendet, um abgelaufene Sitzungsdaten zu löschen. Der Speicher sollte alle Sitzungen löschen, deren letzte Änderungszeit älter als `maxlifetime` ist.
- `close` erfordert keine Operationen, sondern gibt einfach true zurück.
- `open` erfordert keine Operationen, sondern gibt einfach true zurück.

**Ersetzen des Speicher-Treibers**
Nachdem das SessionHandlerInterface implementiert wurde, verwenden Sie die folgende Methode, um den untergeordneten Session-Treiber zu ändern.
```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
``` 
- `$class_name` ist der Name der Klasse des SessionHandlers, die das SessionHandlerInterface implementiert. Wenn ein Namespace vorhanden ist, muss der vollständige Namespace angegeben werden.
- `$config` sind die Parameter des Konstruktors der SessionHandler-Klasse.

**Spezifische Implementierung**
*Bitte beachten Sie, dass diese Klasse MySessionHandler nur zur Veranschaulichung des Prozesses zum Ändern des untergeordneten Sitzungstreiber dient und nicht für den produktiven Einsatz geeignet ist.*
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

// Angenommen, die neue implementierte SessionHandler-Klasse benötigt einige Konfigurationsparameter
$config = ['host' => 'localhost'];
// Verwendung von Workerman\Protocols\Http\Session::handlerClass, um den untergeordneten Session-Treiber zu ändern
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
