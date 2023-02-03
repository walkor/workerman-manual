# workerman-redis

## 介绍

workeman/redis是基于workerman的异步redis组件。

> **注意**
> 此项目主要目的是实现redis异步订阅(subscribe、pSubscribe)
> 因为redis足够快，所以除非需要psubscribe subscribe异步订阅，否则无需使用此异步客户端，使用redis扩展会有更好的性能

## 安装：
```
composer require workerman/redis
```

## 回调用法

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

## 协程用法

> **注意**
> 协程用法需要workerman>=5.0，workerman/redis>=2.0.0 并安装 composer require revolt/event-loop ^1.0.0

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

当不设置回调函数时，客户端会用同步的方式返回异步请求结果，请求过程不阻塞当前进程，也就是可以并发处理请求。

> **注意**
> psubscribe subscribe 不支持协程用法

## 文档
**说明**

**回调方式中，回调函数一般有2个参数($result, $redis)，`$result`为结果，`$redis`为redis实例。例如：**
```
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// 设置回调函数判断set调用结果
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// 回调函数都是可选测参数，这里省略了回调函数
$redis->set('key1', 'value1');
// 回调函数可以嵌套
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

### **连接**
```php
use Workerman\Redis\Client;
// 省略回调
$redis = new Client('redis://127.0.0.1:6379');
// 带回调
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // 设置连接超时10秒，不设置默认5秒
], function ($success, $redis) {
    // 连接结果回调
    if (!$success) echo $redis->error();
});
```

### **auth**
```php
//  密码验证
$redis->auth('password', function ($result) {
    
});
// 用户名密码验证
$redis->auth('username', 'password', function ($result) {

});
```

### **pSubscribe**

订阅一个或多个符合给定模式的频道。

每个模式以 \* 作为匹配符，比如 it\* 匹配所有以 it 开头的频道( it.news 、 it.blog 、 it.tweets 等等)。 news.\* 匹配所有以 news. 开头的频道( news.it 、 news.global.today 等等)，诸如此类。

注意：pSubscribe回调函数有4个参数($pattern, $channel, $message, $redis)

当$redis实例调用pSubscribe或subscribe接口后，当前实例再调用其它方法将被忽略。

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

### **subscribe**

用于订阅给定的一个或多个频道的信息。

注意：subscribe回调函数有3个参数($channel, $message, $redis)

当$redis实例调用pSubscribe或subscribe接口后，当前实例再调用其它方法将被忽略。

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

### **publish**

用于将信息发送到指定的频道。

返回接收到信息的订阅者数量。

```php
$redis2->publish('news', 'news content');
```


### **select**
```php
// 省略回调
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // select参数必须为数字，所以这里$result为false
    var_dump($result, $redis->error());
});
```

### **get**

命令用于获取指定 key 的值。如果 key 不存在，返回 NULL 。如果key 储存的值不是字符串类型，返回false。
```php
$redis->get('key', function($result) {
     // 如果key不存在则返回NULL，发生错误则返回false
    var_dump($result);
});
```

### **set**

用于设置给定 key 的值。如果 key 已经存储其他值， SET 就覆写旧值，且无视类型。
```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// 第三个参数可以传递过期时间，10秒后过期
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

### **setEx, pSetEx**

为指定的 key 设置值及其过期时间。如果 key 已经存在， SETEX 命令将会替换旧的值。

```php
// 注意第二个参数传递过期时间，单位秒
$redis->setEx('key', 3600, 'value'); 
// pSetEx 单位为毫秒
$redis->pSetEx('key', 3600, 'value'); 
```

### **del**

用于删除已存在的键，返回结果为数字，代表删除了多少个key(不存在的key不做计数)。
```php
// 删除一个key
$redis->del('key');
// 删除多个key
$redis->del(['key', 'key1', 'key2']);
```

### **setNx**

（**SET**if**N**ot e**X**ists） 命令在指定的 key 不存在时，为 key 设置指定的值。
```php
$redis->del('key');
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 1
});
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 0
});
```

### **exists**

命令用于检查给定 key 是否存在。返回结果为数字，代表存在的key的个数。
```
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

### **incr, incrBy**

将 key 中储存的数字值增一/制定的值。 如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 incr/incrBy 操作。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回false。
成功则返回为增加后的数值。
```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

### **incrByFloat**

为 key 中所储存的值加上指定的浮点数增量值。
如果 key 不存在，那么 INCRBYFLOAT 会先将 key 的值设为 0 ，再执行加法操作。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回false。
成功则返回增加后的数值。
```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

### **decr, decrBy**

命令将 key 所储存的值减去一/指定的减量值。
如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 decr/decrBy 操作。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回false。
成功则返回减少后的数值。
```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

### **mGet**

返回所有(一个或多个)给定 key 的值。 如果给定的 key 里面，有某个 key 不存在，那么这个 key 返回NULL。
```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```

### **getSet**

用于设置指定 key 的值，并返回 key 的旧值。

```php
$redis->set('x', '42');
$redis->getSet('x', 'lol', function ($result) {
    var_dump($result); // '42'
}) ;
$redis->get('x', function ($result) {
    var_dump($result); // 'lol'
}) ;
```

### **randomKey**

从当前数据库中随机返回一个 key。
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    }) ;
})
```

### **move**

将当前数据库的 key 移动到给定的数据库 db 当中。
```php
$redis->select(0);	// switch to DB 0
$redis->set('x', '42');	// write 42 to x
$redis->move('x', 1, function ($result) { 	// move to DB 1
    var_dump($result); // 1
}) ;  
$redis->select(1);	// switch to DB 1
$redis->get('x', function ($result) {
    var_dump($result); // '42'
}) ;
```

### **rename**

修改 key 的名称，key不存在则返回false。
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
}) ;
```

### **renameNx**

在新的 key 不存在时修改 key 的名称。
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
}) ;
```

### **expire**

于设置 key 的过期时间，key 过期后将不再可用。单位以秒。成功返回1，key不存在返回0，发生错误返回false。
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

### **keys**

命令用于查找所有符合给定模式 pattern 的 key 。
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
}) ;
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
}) ;
```

### **type**

返回 key 所储存的值的类型。返回结果为字符串， string set list zset hash none 中的一种，其中none表示key不存在。
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
}) ;
```

### **append**

如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾，并返回字符串长度。

如果 key 不存在， APPEND 就简单地将给定 key 设为 value ，就像执行 SET key value 一样，并返回字符串长度。

如果key存在，但是不是一个字符串，则返回false。

```php
$redis->set('key', 'value1');
$redis->append('key', 'value2', function ($result) {
    var_dump($result); // 12
}) ; 
$redis->get('key', function ($result) {
    var_dump($result); // 'value1value2'
}) ;
```

### **getRange**

获取存储在指定 key 中字符串的子字符串。字符串的截取范围由 start 和 end 两个偏移量决定(包括 start 和 end 在内)。如果key不存在，则返回空字符串。如果key不是字符串类型，则返回false。
```php
$redis->set('key', 'string value');
$redis->getRange('key', 0, 5, function ($result) {
    var_dump($result); // 'string'
}) ; 
$redis->getRange('key', -5, -1 , function ($result) {
    var_dump($result); // 'value'
}) ;
```

### **setRange**
用指定的字符串覆盖给定 key 所储存的字符串值，覆盖的位置从偏移量 offset 开始。如果key不存在，则将key设置为指定字符串。如果key不是字符串类型，则返回false。

返回结果为更改后的字符串长度。

```php
$redis->set('key', 'Hello world');
$redis->setRange('key', 6, "redis", function ($result) {
    var_dump($result); // 11
}) ; 
$redis->get('key', function ($result) {
    var_dump($result); // 'Hello redis'
}) ; 
```

### **strLen**

获取指定 key 所储存的字符串值的长度。当 key 储存的不是字符串值时，返回false。

```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
}) ; 
```

### **getBit**

对 key 所储存的字符串值，获取指定偏移量上的位(bit)。

```php
$redis->set('key', "\x7f"); // this is 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
}) ; 
```

### **setBit**

对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
返回值为0或1，是修改前的值。

```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2f = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
}) ; 
```

### **bitOp**

在多个键（包含字符串值）之间执行按位操作并将结果存储在目标键中。

BITOP 命令支持四个按位运算：**AND**，**OR**，**XOR**和**NOT**。

返回结果存储在目标密钥中的字符串的大小，即等于最长输入字符串的大小。

```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
}) ;
```

### **bitCount**

计算字符串中的设置位数（人口计数）。

默认情况下，会检查字符串中包含的所有字节。只能在传递附加参数 *start* 和 *end* 的间隔中指定计数操作。

与 GETRANGE 命令类似，开始和结束可以包含负值，以便从字符串的末尾开始索引字节，其中-1是最后一个字节，-2是倒数第二个字符，等等。

结果返回字符串中值为1的位数。

不存在的键被视为空字符串，因此该命令将返回零。

```php
$redis->set('key', 'hello');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
}) ;
$redis->bitCount( 'key', function ($result) {
    var_dump($result); //21
}) ;
```

### **sort**

sort命令可以对list、set和sorted set的元素进行排序。

原型：`sort($key, $options, $callback);`

其中options是以下可选的key和值
~~~
$options = [
     'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // or an array of patterns
    'sort' => 'asc', // or 'desc'
    'alpha' => true,
    'store' => 'external-key'
];
~~~

```php
$redis->del('s');
$redis->sAdd('s', 5);
$redis->sAdd('s', 4);
$redis->sAdd('s', 2);
$redis->sAdd('s', 1);
$redis->sAdd('s', 3);
$redis->sort('s', [], function ($result) {
    var_dump($result); // 1,2,3,4,5
}); 
$redis->sort('s', ['sort' => 'desc'], function ($result) {
    var_dump($result); // 5,4,3,2,1
}); 
$redis->sort('s', ['sort' => 'desc', 'store' => 'out'], function ($result) {
    var_dump($result); // (int)5
}); 
```

### **ttl, pttl**

以秒/毫秒为单位返回 key 的剩余过期时间。

如果key没有ttl，则返回-1。如果key不存在，则返回-2。

```php
$redis->set('key', 'value', 10);
// 秒为单位
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// 毫秒为单位
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// key 不存在
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```

### **persist**

移除给定 key 的过期时间，使得 key 永不过期。

如果成功移除则返回1，如果key不存在或者没有过期时间返回0，发生错误返回false。
```php
$redis->persist('key');
```

### **mSet, mSetNx**

在一个原子命令中设置多个键值对。mSetNx仅在设置了所有键的情况下返回1。

成功返回1，失败返回0，发生错误返回false。
```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

### **hSet**

为哈希表中的字段赋值 。

如果字段是哈希表中的一个新建字段，并且值设置成功，返回 1 。 如果哈希表中域字段已经存在且旧值已被新值覆盖，返回 0 。

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello', function ($r) {
    var_dump($r); // 1
}); 
$redis->hGet('h', 'key1', function ($r) {
    var_dump($r); // hello
}); 
$redis->hSet('h', 'key1', 'plop', function ($r) {
    var_dump($r); // 0
});
$redis->hGet('h', 'key1', function ($r) {
    var_dump($r); // plop
}); 
```

### **hSetNx**

为哈希表中不存在的的字段赋值 。

如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作。

如果字段已经存在于哈希表中，操作无效。

如果 key 不存在，一个新哈希表被创建并执行 HSETNX 命令。

```php
$redis->del('h');
$redis->hSetNx('h', 'key1', 'hello', function ($r) {
    var_dump($r); // 1
});
$redis->hSetNx('h', 'key1', 'world', function ($r) {
    var_dump($r); // 0
});
```

### **hGet**

返回哈希表中指定字段的值。

如果给定的字段或 key 不存在时，返回 null。

```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```


### **hLen**

用于获取哈希表中字段的数量。

当 key 不存在时，返回 0 。

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

### **hDel**

命令用于删除哈希表 key 中的一个或多个指定字段，不存在的字段将被忽略。

返回被成功删除字段的数量，不包括被忽略的字段。如果key不是hash则返回false。

```php
$redis->hDel('h', 'key1');
```

### **hKeys**

以数组的形式获取哈希表中的所有域。

如果key不存在，则返回空的数组。如果key对应的不是hash，则返回false。

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

### **hVals**

以数组的形式返回哈希表所有域的值。

如果key不存在，则返回空的数组。如果key对应的不是hash，则返回false。

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

### **hGetAll**

以关联数组的形式返回哈希表中所有的字段和值。

如果key不存在，则返回空数组。如果key部署hash类型，则返回false。

```php
$redis->del('h');
$redis->hSet('h', 'a', 'x');
$redis->hSet('h', 'b', 'y');
$redis->hSet('h', 'c', 'z');
$redis->hSet('h', 'd', 't');
$redis->hGetAll('h', function ($result) {
    var_export($result); 
});
```
返回
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

### **hExists**

查看哈希表的指定字段是否存在。存在返回1，字段不存在或者key不存在返回0，发生错误返回false。

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

### **hIncrBy**

用于为哈希表中的字段值加上指定增量值。

增量也可以为负数，相当于对指定字段进行减法操作。

如果哈希表的 key 不存在，一个新的哈希表被创建并执行 HINCRBY 命令。

如果指定的字段不存在，那么在执行命令前，字段的值被初始化为 0 。

对一个储存字符串值的字段执行 HINCRBY 命令将返回false。

本操作的值被限制在 64 位(bit)有符号数字表示之内。

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

### **hIncrByFloat**

类似与hIncrBy，只不过增量是浮点型。

### **hMSet**

同时将多个 field-value (字段-值)对设置到哈希表中。

此命令会覆盖哈希表中已存在的字段。

如果哈希表不存在，会创建一个空哈希表，并执行 HMSET 操作。

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

### **hMGet**

以关联数组的形式返回哈希表中一个或多个给定字段的值。

如果指定的字段不存在于哈希表，那么对应的字段将是null值 。如果key不是hash则返回false。

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
输出
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```
### **blPop, brPop**

移出并获取列表的第一个元素/最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');

$redis->blPop(['key1', 'key2'], 10, function ($r) {
    var_export($r); // array ( 0 => 'key1',1 => 'a')
});

Timer::add(1, function () use ($redis2) {
    $redis2->lpush('key1', 'a');
});
```

### **bRPopLPush**

从列表中取出最后一个元素，并插入到另外一个列表的头部； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。如果超时则返回null。

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->del(['key1', 'key2']);
$redis->bRPopLPush('key1', 'key2', 2, function ($r) {
    var_export($r);
});
Timer::add(2, function () use ($redis2) {
    $redis2->lpush('key1', 'a');
    $redis2->lRange('key2', 0, -1, function ($r) {
        var_dump($r);
    });
}, null, false);
```

### **lIndex**

通过索引获取列表中的元素。你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

 如果指定索引值不在列表的区间范围内，返回 null。如果对应的key不是列表类型，则返回false。

```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

### **lInsert**

在列表的元素前或者后插入元素。当指定元素不存在于列表中时，不执行任何操作。

当列表不存在时，被视为空列表，不执行任何操作。

如果 key 不是列表类型则返回false。

```php
$redis->del('key1');
$redis->lInsert('key1', 'after', 'A', 'X', function ($r) {
    var_dump($r); // 0
});
$redis->lPush('key1', 'A');
$redis->lPush('key1', 'B');
$redis->lPush('key1', 'C');
$redis->lInsert('key1', 'before', 'C', 'X', function ($r) {
    var_dump($r); // 4
});
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B', 'X', 'C']
});
```

### **lPop**

移除并返回列表的第一个元素。

当列表 key 不存在时，返回 null 。

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```

### **lPush**

一个或多个值插入到列表头部。 如果 key 不存在，一个空列表会被创建并执行 LPUSH 操作。 当 key 存在但不是列表类型时返回false。

**注意：**在Redis 2.4版本以前的 LPUSH 命令，都只接受单个 value 值。

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

### **lPushx**

将一个值插入到已存在的列表头部，列表不存在时操作无效，返回0。如果key不是列表类型则返回false。

返回值为lPushx 命令执行之后，列表的长度。

```php
$redis->del('key1');
    $redis->lPush('key1', 'A');
$redis->lPushx('key1', ['B','C'], function ($r) {
    var_dump($r); // 3
});
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

### **lRange**

返回列表中指定区间内的元素，区间以偏移量 START 和 END 指定。 其中 0 表示列表的第一个元素， 1 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

返回一个包含指定区间内的元素数组。如果key不是列表类型则返回false。

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

### **lRem**

根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。

COUNT 的值可以是以下几种：

*   count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。
*   count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。
*   count = 0 : 移除表中所有与 VALUE 相等的值。

返回被移除元素的数量。 列表不存在时返回 0。不是列表时返回false。

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

### **lSet**

通过索引来设置元素的值。

成功时返回true，当索引参数超出范围，或对一个空列表进行 LSET 时返回false。

```php
$redis->lSet('key1', 0, 'X');
```

### **lTrim**

对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

下标 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

成功时返回true，失败时返回false。

```php
$redis->del('key1');

$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B', 'C']
});
$redis->lTrim('key1', 0, 1);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B']
});
```

### **rPop**

用于移除列表的最后一个元素，返回值为移除的元素。

当列表不存在时，返回 null 。

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

### **rPopLPush**

于移除列表的最后一个元素，并将该元素添加到另一个列表并返回。

```php
$redis->del('x', 'y');
$redis->lPush('x', 'abc');
$redis->lPush('x', 'def');
$redis->lPush('y', '123');
$redis->lPush('y', '456');
$redis->rPopLPush('x', 'y', function ($r) {
    var_dump($r); // abc
});
$redis->lRange('x', 0, -1, function ($r) {
    var_dump($r); // ['def']
});
$redis->lRange('y', 0, -1, function ($r) {
    var_dump($r); // ['abc', '456', '123']
});
```

### **rPush**

将一个或多个值插入到列表的尾部(最右边)，并返回插入后的列表的长度。

如果列表不存在，一个空列表会被创建并执行 RPUSH 操作。 当列表存在但不是列表类型时返回false。

**注意：**在 Redis 2.4 版本以前的 RPUSH 命令，都只接受单个 value 值。

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

### **rPushX**

将一个值插入到已存在的列表尾部(最右边)并返回列表的长度。如果列表不存在，操作无效，返回0。 当列表存在但不是列表类型时返回false。

````php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

### **lLen**

返回列表的长度。 如果列表 key 不存在，则 key 被解释为一个空列表，返回 0 。 如果 key 不是列表类型，返回false。

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lLen('key1', function ($r) {
    var_dump($r); // 3
});
```

### **sAdd**

将一个或多个成员元素加入到集合中，已经存在于集合的成员元素将被忽略。

假如集合 key 不存在，则创建一个只包含添加的元素作成员的集合。

当集合 key 不是集合类型时返回false。

**注意：**在 Redis2.4 版本以前， SADD 只接受单个成员值。

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , ['member2', 'member3'], function ($r) {
    var_dump($r); // 2
});
$redis->sAdd('key1' , 'member2', function ($r) {
    var_dump($r); // 0
});
```

### **sCard**

返回集合中元素的数量。当集合 key 不存在时返回 0 。

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , 'member2');
$redis->sAdd('key1' , 'member3');
$redis->sCard('key1', function ($r) {
    var_dump($r); // 3
});
$redis->sCard('keyX', function ($r) {
    var_dump($r); // 0
});
```

### **sDiff**

返回第一个集合与其他集合之间的差异，也可以认为说第一个集合中独有的元素。不存在的集合 key 将视为空集。

```php
$redis->del('s0', 's1', 's2');

$redis->sAdd('s0', '1');
$redis->sAdd('s0', '2');
$redis->sAdd('s0', '3');
$redis->sAdd('s0', '4');
$redis->sAdd('s1', '1');
$redis->sAdd('s2', '3');
$redis->sDiff(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // ['2', '4']
});
```

### **sDiffStore**

给定集合之间的差集存储在指定的集合中。如果指定的集合 key 已存在，则会被覆盖。

```php
$redis->del('s0', 's1', 's2');
$redis->sAdd('s0', '1');
$redis->sAdd('s0', '2');
$redis->sAdd('s0', '3');
$redis->sAdd('s0', '4');
$redis->sAdd('s1', '1');
$redis->sAdd('s2', '3');
$redis->sDiffStore('dst', ['s0', 's1', 's2'], function ($r) {
    var_dump($r); // 2
});
$redis->sMembers('dst', function ($r) {
    var_dump($r); // ['2', '4']
});
```

### **sInter**

返回给定所有给定集合的交集。 不存在的集合 key 被视为空集。 当给定集合当中有一个空集时，结果也为空集。

```php
$redis->del('s0', 's1', 's2');
$redis->sAdd('key1', 'val1');
$redis->sAdd('key1', 'val2');
$redis->sAdd('key1', 'val3');
$redis->sAdd('key1', 'val4');
$redis->sAdd('key2', 'val3');
$redis->sAdd('key2', 'val4');
$redis->sAdd('key3', 'val3');
$redis->sAdd('key3', 'val4');
$redis->sInter(['key1', 'key2', 'key3'], function ($r) {
    var_dump($r); // ['val4', 'val3']
});
```

### **sInterStore**

将给定集合之间的交集存储在指定的集合中并返回存储交集的集合的元素数量。如果指定的集合已经存在，则将其覆盖。

```php
$redis->sAdd('key1', 'val1');
$redis->sAdd('key1', 'val2');
$redis->sAdd('key1', 'val3');
$redis->sAdd('key1', 'val4');

$redis->sAdd('key2', 'val3');
$redis->sAdd('key2', 'val4');

$redis->sAdd('key3', 'val3');
$redis->sAdd('key3', 'val4');

$redis->sInterStore('output', 'key1', 'key2', 'key3', function ($r) {
    var_dump($r); // 2
});
$redis->sMembers('output', function ($r) {
    var_dump($r); // ['val4', 'val3']
});
```

### **sIsMember**

判断成员元素是否是集合的成员。

如果成员元素是集合的成员返回 1 。 如果成员元素不是集合的成员，或 key 不存在返回 0 。如果key对应的不是集合类型，则返回false。

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

### **sMembers**

返回集合中的所有的成员。 不存在的集合 key 被视为空集合。

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

### **sMove**

将指定成员 member 元素从 source 集合移动到 destination 集合。

SMOVE 是原子性操作。

如果 source 集合不存在或不包含指定的 member 元素，则 SMOVE 命令不执行任何操作，仅返回 0 。否则， member 元素从 source 集合中被移除，并添加到 destination 集合中去。

当 destination 集合已经包含 member 元素时， SMOVE 命令只是简单地将 source 集合中的 member 元素删除。

当 source 或 destination 不是集合类型时返回false。

```php
$redis->sMove('key1', 'key2', 'member13');
```

### **sPop**

移除集合中的指定 key 的一个或多个随机元素，移除后会返回移除的元素。

当集合不存在或是空集时，返回 null 。

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , 'member2');
$redis->sAdd('key1' , 'member3');
$redis->sPop('key1', function ($r) {
    var_dump($r); // member3
});
$redis->sAdd('key2', ['member1', 'member2', 'member3']);
$redis->sPop('key2', 3, function ($r) {
    var_dump($r); // ['member1', 'member2', 'member3']
});
```

### **sRandMember**

Redis Srandmember 命令用于返回集合中的一个随机元素。

从 Redis 2.6 版本开始， Srandmember 命令接受可选的 count 参数：

*   如果 count 为正数，且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合。
*   如果 count 为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为 count 的绝对值。

该操作和 SPOP 相似，但 SPOP 将随机元素从集合中移除并返回，而 Srandmember 则仅仅返回随机元素，而不对集合进行任何改动。

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , 'member2');
$redis->sAdd('key1' , 'member3'); 

$redis->sRandMember('key1', function ($r) {
    var_dump($r); // member1
});

$redis->sRandMember('key1', 2, function ($r) {
    var_dump($r); // ['member1', 'member2']
});
$redis->sRandMember('key1', -100, function ($r) {
    var_dump($r); // ['member1', 'member2', 'member3', 'member3', ...]
});
$redis->sRandMember('empty-set', 100, function ($r) {
    var_dump($r); // []
}); 
$redis->sRandMember('not-a-set', 100, function ($r) {
    var_dump($r); // []
});
```

### **sRem**

移除集合中的一个或多个成员元素，不存在的成员元素会被忽略。

返回被成功移除的元素的数量，不包括被忽略的元素。

当 key 不是集合类型返回false。

在 Redis 2.4 版本以前， SREM 只接受单个成员值。

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

### **sUnion**

命令返回给定集合的并集。不存在的集合 key 被视为空集。

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

### **sUnionStore**

将给定集合的并集存储在指定的集合 destination 中并返回元素数量。如果 destination 已经存在，则将其覆盖。

```php
$redis->del('s0', 's1', 's2');
$redis->sAdd('s0', '1');
$redis->sAdd('s0', '2');
$redis->sAdd('s1', '3');
$redis->sAdd('s1', '1');
$redis->sAdd('s2', '3');
$redis->sAdd('s2', '4');
$redis->sUnionStore('dst', 's0', 's1', 's2', function ($r) {
    var_dump($r); // 4
});
$redis->sMembers('dst', function ($r) {
    var_dump($r); // ['1', '2', '3', '4']
});
```

****





