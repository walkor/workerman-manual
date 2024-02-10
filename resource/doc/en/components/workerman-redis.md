# workerman-redis

## Introduction

workeman/redis is an asynchronous Redis component based on Workerman.

> **Note**
> The main purpose of this project is to implement asynchronous subscription (subscribe, pSubscribe) for Redis.
> Because Redis is fast enough, it is unnecessary to use this asynchronous client unless asynchronous subscription for pSubscribe and subscribe is needed. Using the Redis extension will provide better performance.

## Installation:
```shell
composer require workerman/redis
```

## Callback Usage

```php
use Workerman\Worker;
use Workerman\Redis\Client;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global $redis;
    $redis = new Client('redis://127.0.0.1:6379');
};

$worker->onMessage = function(TcpConnection $connection, $data) {
    global $redis;
    $redis->set('key', 'hello world');    
    $redis->get('key', function ($result) use ($connection) {
        $connection->send($result);
    });  
};

Worker::runAll();
```

## Coroutine Usage

> **Note**
> The coroutine usage requires workerman>=5.0, workerman/redis>=2.0.0, and the installation of composer require revolt/event-loop ^1.0.0

```php
use Workerman\Worker;
use Workerman\Redis\Client;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global $redis;
    $redis = new Client('redis://127.0.0.1:6379');
};

$worker->onMessage = function(TcpConnection $connection, $data) {
    global $redis;
    $redis->set('key', 'hello world');    
    $result = $redis->get('key');
    $connection->send($result);
};

Worker::runAll();
```

When no callback function is set, the client will return the asynchronous request result in a synchronous way. The request process does not block the current process, allowing concurrent request processing.

> **Note**
> psubscribe subscribe does not support coroutine usage.
## Documentation
**Instructions**
**In the callback method, there are generally 2 parameters ($result, $redis), `$result` is the result, and `$redis` is the Redis instance. For example:**
```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// Setting a callback function to evaluate set call result
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// Callback functions are all optional parameters, and are omitted here
$redis->set('key1', 'value1');
// Callback functions can be nested
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

### **Connect**
```php
use Workerman\Redis\Client;
// Omit the callback
$redis = new Client('redis://127.0.0.1:6379');
// With callback
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // Set connect timeout to 10 seconds, default is 5 seconds if not set
], function ($success, $redis) {
    // Callback for connect result
    if (!$success) echo $redis->error();
});
```

### **Auth**
```php
// Password validation
$redis->auth('password', function ($result) {
    
});
// Username and password validation
$redis->auth('username', 'password', function ($result) {

});
```

### **PSubscribe**
Subscribe to one or more channels based on the given pattern.
Each pattern uses \* as a matching character. For example, it\* matches all channels starting with it (it.news, it.blog, it.tweets, and so on). news.\* matches all channels starting with news. (news.it, news.global.today, etc.), and so on.

Note: The callback function for pSubscribe has 4 parameters ($pattern, $channel, $message, $redis).
After the $redis instance calls the pSubscribe or subscribe interface, subsequent method calls on the current instance will be ignored.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->psubscribe(['news*', 'blog*'], function ($pattern, $channel, $message) {
    echo "$pattern, $channel, $message"; // news*, news.add, news content
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news.add', 'news content');
});
```

### **Subscribe**
Used to subscribe to information from one or more specified channels.
Note: The subscribe callback function has 3 parameters ($channel, $message, $redis).
After the $redis instance calls the pSubscribe or subscribe interface, subsequent method calls on the current instance will be ignored.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->subscribe(['news', 'blog'], function ($channel, $message) {
    echo "$channel, $message"; // news, news content
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news', 'news content');
});
```

### **Publish**
Used to send information to the specified channel.

Returns the number of subscribers that received the information.

```php
$redis2->publish('news', 'news content');
```

### **Select**
```php
// Omit the callback
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // The select parameter must be a number, so $result here is false
    var_dump($result, $redis->error());
});
```

### **Get**
The command is used to get the value of the specified key. If the key does not exist, it returns NULL. If the key's value is not a string, it returns false.

```php
$redis->get('key', function($result) {
     // If the key does not exist, it returns NULL; if an error occurs, it returns false
    var_dump($result);
});
```

### **Set**
Used to set the value of the given key. If the key already holds a value, the SET command overwrites the old value, regardless of its type.

```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// The third parameter can pass the expiration time, expiring after 10 seconds
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

### **SetEx, PSetEx**
Set the value and its expiration time for the specified key. If the key already exists, the SETEX command will replace the old value.

```php
// Note that the second parameter passes the expiration time in seconds
$redis->setEx('key', 3600, 'value'); 
// pSetEx in milliseconds
$redis->pSetEx('key', 3600, 'value'); 
```

### **Del**
Used to delete existing keys, and the result is the number of keys deleted (non-existent keys are not counted).

```php
// Delete one key
$redis->del('key');
// Delete multiple keys
$redis->del(['key', 'key1', 'key2']);
```

### **SetNx**
(SET if Not exists) command sets the key to hold the value if it doesn't exist. If the key already exists, it returns 0.

```php
$redis->del('key');
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 1
});
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 0
});
```

### **Exists**
The EXISTS command is used to check whether the given key exists. The result is the number of existing keys.

```php
$redis->set('key', 'value');
$redis->exists('key', function ($result) {
    var_dump($result); // 1
}); 
$redis->exists('NonExistingKey', function ($result) {
    var_dump($result); // 0
}); 
$redis->mset(['foo' => 'foo', 'bar' => 'bar', 'baz' => 'baz']);
$redis->exists(['foo', 'bar', 'baz'], function ($result) {
    var_dump($result); // 3
}); 
```

### **Incr, IncrBy**
Increments the number stored at key by one/increments it by the specified value. If the key does not exist, it's set to 0 before performing the operation.

```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

### **IncrByFloat**
Increments the number stored at key by the specified floating-point incremental value.

```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

### **Decr, DecrBy**
Decrements the number stored at key. If the key does not exist, it is set to 0 before performing the operation.

```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```
...
