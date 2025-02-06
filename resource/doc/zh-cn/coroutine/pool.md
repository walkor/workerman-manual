# Pool 连接池

多个协程共用同一个连接会导致数据混乱，所以需要使用连接池来管理数据库、redis等连接资源。

> **提示**
> 此特性需要 workerman>=5.1.0

## 注意
* 底层自动支持Swoole/Swow/Fiber/Select/Event驱动
* 当使用Fiber/Select/Event驱动时，如果使用的是PDO redis等阻塞式扩展，则自动退化为只有一个连接的连接池

## Redis连接池

```php
<?php
use Workerman\Connection\TcpConnection;
use Workerman\Coroutine\Pool;
use Workerman\Events\Swoole;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

class RedisPool
{

    private Pool $pool;
    
    public function __construct($host, $port, $max_connections = 10)
    {
        $this->pool = new Pool($max_connections);
        // 设置连接池创建连接的方法
        $this->pool->setConnectionCreator(function () use ($host, $port) {
            $redis = new \Redis();
            $redis->connect($host, $port);
            return $redis;
        });
        // 设置连接池销毁连接的方法
        $this->pool->setConnectionCloser(function ($redis) {
            $redis->close();
        });
        // 设置心跳检测方法
        $this->pool->setHeartbeatChecker(function ($redis) {
            $redis->ping();
        });
    }
    
    // 获取连接
    public function get(): \Redis
    {
        return $this->pool->get();
    }
    
    // 归还连接
    public function put($redis): void
    {
        $this->pool->put($redis);
    }
}

// Http Server
$worker = new Worker('http://0.0.0.0:8001');
$worker->eventLoop = Swoole::class; // Or Swow::class or Fiber::class
$worker->onMessage = function (TcpConnection $connection, Request $request) {
    static $pool;
    if (!$pool) {
        $pool = new RedisPool('127.0.0.1', 6379, 10);
    }
    $redis = $pool->get();
    $redis->set('key', 'hello');
    $value = $redis->get('key');
    $pool->put($redis);
    $connection->send($value);
};

Worker::runAll();
```

## MySQL连接池(支持自动获取和归还连接)
```php
<?php
use Workerman\Connection\TcpConnection;
use Workerman\Coroutine\Context;
use Workerman\Coroutine;
use Workerman\Coroutine\Pool;
use Workerman\Events\Swoole;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

class Db
{
    private static ?Pool $pool = null;
    
    public static function __callStatic($name, $arguments)
    {
        if (self::$pool === null) {
            self::initializePool();
        }
        // 从协程上下文中获取连接，保证同一个协程使用同一个连接
        $pdo = Context::get('pdo');
        if (!$pdo) {
            // 从连接池中获取连接
            $pdo = self::$pool->get();
            Context::set('pdo', $pdo);
            // 当协程结束时，自动归还连接
            Coroutine::defer(function () use ($pdo) {
                self::$pool->put($pdo);
            });
        }
        return call_user_func_array([$pdo, $name], $arguments);
    }
    
    private static function initializePool(): void
    {
        self::$pool = new Pool(10);
        self::$pool->setConnectionCreator(function () {
            return new \PDO('mysql:host=127.0.0.1;dbname=your_database', 'your_username', 'your_password');
        });
        self::$pool->setConnectionCloser(function ($pdo) {
            $pdo = null;
        });
        self::$pool->setHeartbeatChecker(function ($pdo) {
            $pdo->query('SELECT 1');
        });
    }
    
}

// Http Server
$worker = new Worker('http://0.0.0.0:8001');
$worker->eventLoop = Swoole::class; // Or Swow::class or Fiber::class
$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $value = Db::query('SELECT NOW() as now')->fetchAll();
    $connection->send(json_encode($value));
};

Worker::runAll();
```

## 接口说明

```php
interface PoolInterface
{

    /**
     * 构造函数
     * @param int $max_connections 最大连接数，默认1
     * @param array $config = [
     *    'min_connections' => 1, // 最小连接数，默认为1
     *    'idle_timeout' => 60, // 连接空闲超时时间(秒)，默认为60秒，超过60秒后连接会被销毁从连接池中移除
     *    'heartbeat_interval => 50, // 心跳检测间隔时间(秒)，默认为50秒，每隔50秒会检测一次连接是否正常
     *    'wait_timeout' => 10, // 获取连接的等待超时时间(秒)，默认为10秒，超过10秒获取连接失败抛出异常
     * ] 
     */
    public function __construct(int $max_connections = 1, array $config = []);

    /**
     * 获取一个连接
     */
    public function get(): mixed;

    /**
     * 归还一个连接
     */
    public function put(object $connection): void;

    /**
     * 创建一个连接
     */
    public function createConnection(): object;

    /**
     * 关闭连接，并从连接池中移除
     */
    public function closeConnection(object $connection): void;

    /**
     * 获取连接池中当前的连接数(包括已获取在使用中的连接和未获取的连接)
     */
    public function getConnectionCount(): int;

    /**
     * 关闭连接池中的连接(不包括已获取的在使用中的连接)
     */
    public function closeConnections(): void;

    /**
     * 设置连接池创建连接的方法
     */
    public function setConnectionCreator(callable $connectionCreateHandler): self
    
    /**
     * 设置连接池销毁连接的方法
     */
    public function setConnectionCloser(callable $connectionDestroyHandler): self
    
    /**
     * 设置心跳检测方法
     */
    public function setHeartbeatChecker(callable $connectionHeartbeatHandler): self
    
}
```