# 说明
workerman从4.x版本开始加强了HTTP服务的支持。引入了请求类、响应类、session类以及[SSE](SSE.md)。如果你想使用workerman的HTTP服务，强烈推荐使用workerman4.x或者以后的更高版本。

**注意以下都是workerman4.x版本的用法，不兼容workerman3.x。**

## 更改session存储引擎
workerman为session提供了文件存储引擎和redis存储引擎。默认使用文件存储引擎。如果想更改为redis存储引擎，请参考如下代码。
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// redis配置
$config = [
    'host'     => '127.0.0.1', // 必选参数
    'port'     => 6379,        // 必选参数
    'timeout'  => 2,           // 可选参数
    'auth'     => '******',    // 可选参数
    'database' => 1,           // 可选参数
    'prefix'   => 'session_'   // 可选参数
];
// 使用 Workerman\Protocols\Http\Session::handlerClass方法来更改session底层驱动类
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## 设置session存储位置
使用默认存储引擎时session数据默认存储在磁盘中，默认位置为`session_save_path()`的返回的位置。
你可以使用以下方法改变存储位置。

```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// 设置session文件存储位置
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// 运行worker
Worker::runAll();
```

## session文件清理
使用默认session存储引擎时磁盘上会有多个session文件，
workerman 会根据php.ini中设置的`session.gc_probability` `session.gc_divisor` `session.gc_maxlifetime` 选项清理过期的session文件。关于这三个选项说明参见 [php手册](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability)

## 更改存储驱动
除了文件session存储引擎和redis session存储引擎，workerman允许你通过标准的[SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) 接口添加新的session存储引擎，比如mangoDb session存储引擎、MySQL session存储引擎等。

**添加新的session存储引擎流程 **
  1.  实现 [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) 接口
  2. 使用 `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` 方法替换底层SessionHandler接口
 
** 实现 SessionHandlerInterface 接口 **

自定义session存储驱动须实现 SessionHandlerInterface 接口。这个接口包含以下方法：
```php
SessionHandlerInterface {
    /* Methods */
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**SessionHandlerInterface 说明**
 - read方法用来从存储中读取session_id对应的所有session数据。请不要对数据进行反序列化操作，框架会自动完成。
 - write方法用来向存储写入session_id对应的session数据。请不要对数据进行序列化操作，框架已经自动完成。
 - destroy方法用来销毁session_id对应的session数据。
 - gc方法用来删除过期的session数据，存储应该对最后修改时间大于maxlifetime的所有session执行删除操作
 - close 无需任何操作，直接返回true即可
 - open 无需任何操作，直接返回true接口

**替换底层驱动**

实现完SessionHandlerInterface接口后，使用以下方法更改session底层驱动。

```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name 为实现SessionHandlerInterface接口的SessionHandler类的名字。如果有命名空间则需要带上完整的命名空间
 - $config 为SessionHandler类的构造函数的参数

**具体实现**

*注意，这个MySessionHandler类仅仅为了说明更改session底层驱动的流程，MySessionHandler并不能用于生产环境。*
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

//  假设新实现的SessionHandler类需要一些配置传入
$config = ['host' => 'localhost'];
//  使用 Workerman\Protocols\Http\Session::handlerClass($class_name, $config) 来更改session底层驱动类
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```




 

