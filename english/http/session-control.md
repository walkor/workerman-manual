# Explanation
Starting from version 4.x, workerman has strengthened the support for HTTP services. It has introduced request class, response class, session class, and [SSE](SSE.md). If you want to use workerman's HTTP service, it is strongly recommended to use workerman 4.x or higher versions.

**Please note that the following are all usage of workerman 4.x and are not compatible with workerman 3.x.**

## Changing the Session Storage Engine
Workerman provides file storage engine and Redis storage engine for sessions. The default is to use the file storage engine. If you want to change to the Redis storage engine, please refer to the following code.
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// redis configuration
$config = [
    'host'     => '127.0.0.1', // required
    'port'     => 6379,        // required
    'timeout'  => 2,           // optional
    'auth'     => '******',    // optional
    'database' => 1,           // optional
    'prefix'   => 'session_'   // optional
];
// Use the Session::handlerClass method to change the underlying session driver class
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## Setting the Session Storage Location
When using the default storage engine, session data is stored in the disk by default, at the location returned by `session_save_path()`. You can change the storage location using the following method.
```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Set the session file storage location
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Run the worker
Worker::runAll();
```

## Session File Cleanup
When using the default session storage engine, there will be multiple session files on the disk. Workerman will clean up expired session files based on the `session.gc_probability`, `session.gc_divisor`, and `session.gc_maxlifetime` options set in php.ini. For more information about these three options, refer to the [php manual](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability).

## Changing the Storage Driver
In addition to the file session storage engine and the Redis session storage engine, workerman allows you to add new session storage engines using the standard [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) interface, such as mangoDb session storage engine, MySQL session storage engine, etc.

**Process for Adding a New Session Storage Engine**
  1. Implement the [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) interface
  2. Use the `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` method to replace the underlying SessionHandler interface

**Implementing the SessionHandlerInterface Interface**

Custom session storage drivers must implement the SessionHandlerInterface interface. This interface includes the following methods:
```php
SessionHandlerInterface {
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**SessionHandlerInterface Explanation**
 - The read method is used to read all session data corresponding to the session_id from the storage. Please do not perform deserialization operations on the data, as the framework will handle it automatically.
 - The write method is used to write session data corresponding to session_id to the storage. Please do not perform serialization operations on the data, as the framework has already handled it.
 - The destroy method is used to destroy the session data corresponding to session_id.
 - The gc method is used to delete expired session data, and the storage should perform deletion operations on all sessions whose last modification time is greater than maxlifetime.
 - The close method requires no operation, simply return true.
 - The open method requires no operation, simply return true.

**Replacing the Underlying Driver**

After implementing the SessionHandlerInterface interface, use the following method to change the underlying session driver.
```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name is the name of the SessionHandler class that implements the SessionHandlerInterface interface. If it has a namespace, the full namespace should be included.
 - $config is the parameter for the constructor of the SessionHandler class.

**Specific Implementation**

*Note that the MySessionHandler class is only used to illustrate the process of changing the underlying session storage driver, and MySessionHandler cannot be used in a production environment.*
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

//  Assuming the newly implemented SessionHandler class needs some configuration input
$config = ['host' => 'localhost'];
//  Use the Workerman\Protocols\Http\Session::handlerClass($class_name, $config) method to change the underlying session driver class
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
