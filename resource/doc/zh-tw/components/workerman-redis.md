# workerman-redis

## 介紹

workman/redis 是基於 workerman 的非同步 redis 組件。

> **注意**
> 此專案主要目的是實現 redis 非同步訂閱 (subscribe、pSubscribe)。
> 由於 redis 非常快速，因此除非需要 psubscribe subscribe 非同步訂閱，否則無需使用此非同步客戶端，使用 redis 擴展會有更好的性能。

## 安裝：

```php
composer require workerman/redis
```

## 回調用法

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

## 協程用法

> **注意**
> 協程用法需要 workerman>=5.0，workerman/redis>=2.0.0 並安裝 composer require revolt/event-loop ^1.0.0

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

當不設置回調函數時，客戶端會以同步方式返回非同步請求結果，請求過程不會阻塞當前進程，也就是可以並發處理請求。

> **注意**
> psubscribe subscribe 不支持協程用法

# 文檔

**說明**

**在回調方式中，回調函數一般有2個參數 ($result, $redis)，`$result` 為結果，`$redis` 為 redis 實例。例如：**
```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// 設置回調函數判斷 set 調用結果
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// 回調函數都是可選測參數，這裡省略了回調函數
$redis->set('key1', 'value1');
// 回調函數可以嵌套
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **連接**
```php
use Workerman\Redis\Client;
// 省略回調
$redis = new Client('redis://127.0.0.1:6379');
// 帶回調
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // 設置連接超時10秒，不設置默認5秒
], function ($success, $redis) {
    // 連接結果回調
    if (!$success) echo $redis->error();
});
```

## **auth**
```php
//  密碼驗證
$redis->auth('password', function ($result) {
    
});
// 使用者名密碼驗證
$redis->auth('username', 'password', function ($result) {

});
```

## **pSubscribe**

訂閱一個或多個符合給定模式的頻道。

每個模式以 \* 作為匹配符，例如 it\* 匹配所有以 it 開頭的頻道（ it.news 、 it.blog 、 it.tweets 等等）。 news.\* 匹配所有以 news. 開頭的頻道（ news.it 、 news.global.today 等等），諸如此類。

注意：pSubscribe 回調函數有4個參數 ($pattern, $channel, $message, $redis)。

當 $redis 實例調用 pSubscribe 或 subscribe 接口後，當前實例再調用其它方法將被忽略。

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

## **subscribe**

用於訂閱給定的一個或多個頻道的信息。

注意：subscribe 回調函數有3個參數 ($channel, $message, $redis)。

當 $redis 實例調用 pSubscribe 或 subscribe 接口後，當前實例再調用其它方法將被忽略。

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

## **publish**

用於將信息發送到指定的頻道。

返回接收到信息的訂閱者數量。

```php
$redis2->publish('news', 'news content');
```


## **select**
```php
// 省略回調
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // select參數必須為數字，所以這裡$result為false
    var_dump($result, $redis->error());
});
```

## **get**

命令用於獲取指定 key 的值。如果 key 不存在，返回 NULL 。如果 key 儲存的值不是字串類型，返回 false。

```php
$redis->get('key', function($result) {
     // 如果key不存在則返回NULL，發生錯誤則返回false
    var_dump($result);
});
```

## **set**

用於設置給定 key 的值。如果 key 已經儲存其他值， SET 就覆蓋舊值，且無視類型。

```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// 第三個參數可以傳遞過期時間，10秒後過期
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **setEx, pSetEx**

為指定的 key 設置值及其過期時間。如果 key 已經存在， SETEX 命令將會替換舊的值。

```php
// 注意第二個參數傳遞過期時間，單位秒
$redis->setEx('key', 3600, 'value'); 
// pSetEx 單位為毫秒
$redis->pSetEx('key', 3600, 'value'); 
```

## **del**

用於刪除已存在的鍵，返回結果為數字，代表刪除了多少個 key（不存在的 key 不做計數）。

```php
// 刪除一個key
$redis->del('key');
// 刪除多個key
$redis->del(['key', 'key1', 'key2']);
```

## **setNx**

（**SET**if**N**ot e**X**ists） 命令在指定的 key 不存在時，為 key 設置指定的值。

```php
$redis->del('key');
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 1
});
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 0
});
```

## **exists**

命令用於檢查給定 key 是否存在。返回結果為數字，代表存在的 key 的個數。

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

## **incr, incrBy**

將 key 中儲存的數字值增一/指定的值。如果 key 不存在，那麼 key 的值會先被初始化為 0 ，然後再執行 incr/incrBy 操作。

如果值包含錯誤的類型，或字串類型的值不能表示為數字，那麼返回 false。成功則返回為增加後的數值。

```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **incrByFloat**

為 key 中所儲存的值加上指定的浮點數增量值。如果 key 不存在，那麼 INCRBYFLOAT 會先將 key 的值設為 0 ，再執行加法操作。

如果值包含錯誤的類型，或字串類型的值不能表示為數字，那麼返回 false。成功則返回增加後的數值。

```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## **decr, decrBy**

命令將 key 所儲存的值減去一/指定的減量值。如果 key 不存在，那麼 key 的值會先被初始化為 0 ，然後再執行 decr/decrBy 操作。

如果值包含錯誤的類型，或字串類型的值不能表示為數字，那麼返回 false。成功則返回減少後的數值。

```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

返回所有（一個或多個）給定 key 的值。如果給定的 key 里面，有某個 key 不存在，那麼這個 key 返回 NULL。

```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```
## **getSet**

用於設置指定 key 的值，並返回 key 的舊值。

```php
$redis->set('x', '42');
$redis->getSet('x', 'lol', function ($result) {
    var_dump($result); // '42'
}) ;
$redis->get('x', function ($result) {
    var_dump($result); // 'lol'
}) ;
```

## **randomKey**

從當前數據庫中隨機返回一個 key。
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    }) ;
})
```

## **move**

將當前數據庫的 key 移動到給定的數據庫 db 中。
```php
$redis->select(0);	// 切換到 DB 0
$redis->set('x', '42');	// 將 42 寫入 x
$redis->move('x', 1, function ($result) { 	// 移動到 DB 1
    var_dump($result); // 1
}) ;  
$redis->select(1);	// 切換到 DB 1
$redis->get('x', function ($result) {
    var_dump($result); // '42'
}) ;
```

## **rename**

修改 key 的名稱，若 key 不存在則返回 false。
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
}) ;
```

## **renameNx**

在新的 key 不存在時修改 key 的名稱。
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
}) ;
```

## **expire**

設置 key 的過期時間，過期後將無法使用。單位為秒。成功返回1，key 不存在返回0，發生錯誤返回 false。
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

命令用於查找所有符合給定模式 pattern 的 key 。
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
}) ;
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
}) ;
```

## **type**

返回 key 儲存的值的類型。返回結果為字符串， string set list zset hash none 中的一種，其中 none 表示 key 不存在。
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
}) ;
```

## **append**

如果 key 已經存在並且是一個字符串，APPEND 命令將 value 追加到 key 原來的值的末尾，並返回字符串長度。

如果 key 不存在，APPEND 就簡單地將給定 key 設為 value ，就像執行 SET key value 一樣，並返回字符串長度。

如果 key 存在，但是不是一個字符串，則返回 false。
```php
$redis->set('key', 'value1');
$redis->append('key', 'value2', function ($result) {
    var_dump($result); // 12
}) ; 
$redis->get('key', function ($result) {
    var_dump($result); // 'value1value2'
}) ;
```

## **getRange**

獲取儲存在指定 key 中字符串的子字符串。字符串的截取範圍由 start 和 end 兩個偏移量決定(包括 start 和 end 在內)。如果 key 不存在，則返回空字符串。如果 key 不是字符串類型，則返回 false。
```php
$redis->set('key', 'string value');
$redis->getRange('key', 0, 5, function ($result) {
    var_dump($result); // 'string'
}) ; 
$redis->getRange('key', -5, -1 , function ($result) {
    var_dump($result); // 'value'
}) ;
```

## **setRange**
用指定的字符串覆蓋給定 key 所儲存的字符串值，覆蓋的位置從偏移量 offset 開始。如果 key 不存在，則將 key 設置為指定字符串。如果 key 不是字符串類型，則返回 false。

返回結果為更改後的字符串長度。
```php
$redis->set('key', 'Hello world');
$redis->setRange('key', 6, "redis", function ($result) {
    var_dump($result); // 11
}) ; 
$redis->get('key', function ($result) {
    var_dump($result); // 'Hello redis'
}) ; 
```

## **strLen**

獲取指定 key 所儲存的字符串值的長度。當 key 儲存的不是字符串值時，返回 false。
```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
}) ; 
```

## **getBit**

對 key 所儲存的字符串值，獲取指定偏移量上的位(bit)。
```php
$redis->set('key', "\x7f"); // this is 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **setBit**

對 key 所儲存的字符串值，設定或清除指定偏移量上的位(bit)。
返回值為 0 或 1，是修改前的值。
```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2a = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **bitOp**

在多個鍵（包含字符串值）之間執行按位操作並將結果儲存在目標鍵中。

BITOP 命令支持四個按位運算：AND，OR，XOR和NOT。

返回結果儲存在目標金鑰中的字符串的大小，即等於最長輸入字符串的大小。
```php
$redis->set('key1', "abc");
$redis->bitOp('AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
}) ;
```

## **bitCount**

計算字符串中設置位數（人口計數）。

默認情況下，將檢查字符串中包含的所有位元組。只能在傳遞附加參數 start 和 end 的間隔中指定計數操作。

與 GETRANGE 命令類似，開始和結束可以包含負值，以便從字符串的末尾開始索引位元組，其中-1是最後一個位元組，-2是倒數第二位位元組，等等。

結果返回字符串中值為1的位數。不存在的鍵被視為空字符串，因此該命令將返回零。
```php
$redis->set('key', 'hello');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
}) ;
$redis->bitCount( 'key', function ($result) {
    var_dump($result); //21
}) ;
```

## **sort**

sort命令可以對list、set和sorted set的元素進行排序。

原型：`sort($key, $options, $callback);`

其中options是以下可選的key和值
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

## **ttl, pttl**

以秒/毫秒為單位返回 key 的剩餘過期時間。

如果key没有ttl，则返回-1。如果key不存在，则返回-2。

```php
$redis->set('key', 'value', 10);
// 秒為單位
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// 毫秒為單位
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// key 不存在
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```

## **persist**

移除給定 key 的過期時間，使得 key 永不過期。

如果成功移除則返回1，如果key不存在或者沒有過期時間返回0，發生錯誤返回false。
```php
$redis->persist('key');
```

## **mSet, mSetNx**

在一個原子命令中設置多個鍵值對。mSetNx僅在設置了所有鍵的情況下返回1。

成功返回1，失敗返回0，發生錯誤返回false。
```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

為哈希表中的欄位賦值 。

如果欄位是哈希表中的一個新建欄位，並且值設置成功，返回 1 。 如果哈希表中域欄位已經存在且舊值已被新值覆蓋，返回 0 。

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

## **hSetNx**

為哈希表中不存在的的欄位賦值 。

如果哈希表不存在，一個新的哈希表被創建並進行 HSET 操作。

如果欄位已經存在於哈希表中，操作無效。

如果 key 不存在，一個新哈希表被創建並執行 HSETNX 命令。

```php
$redis->del('h');
$redis->hSetNx('h', 'key1', 'hello', function ($r) {
    var_dump($r); // 1
});
$redis->hSetNx('h', 'key1', 'world', function ($r) {
    var_dump($r); // 0
});
```

## **hGet**

返回哈希表中指定欄位的值。

如果給定的欄位或 key 不存在時，返回 null。

```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```
## **hLen**

用於獲取哈希表中字段的數量。

當 key 不存在時，返回 0 。

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

命令用於刪除哈希表 key 中的一個或多個指定字段，不存在的字段將被忽略。

返回被成功刪除字段的數量，不包括被忽略的字段。如果key不是hash則返回false。

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

以陣列的形式獲取哈希表中的所有域。

如果key不存在，則返回空的陣列。如果key對應的不是hash，則返回false。

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

以陣列的形式返回哈希表所有域的值。

如果key不存在，則返回空的陣列。如果key對應的不是hash，則返回false。

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

以關聯陣列的形式返回哈希表中所有的字段和值。

如果key不存在，則返回空陣列。如果key部署hash類型，則返回false。

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

## **hExists**

查看哈希表的指定字段是否存在。存在返回1，字段不存在或者key不存在返回0，發生錯誤返回false。

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## **hIncrBy**

用於為哈希表中的字段值加上指定增量值。

增量也可以為負數，相當於對指定字段進行減法操作。

如果哈希表的 key 不存在，一個新的哈希表被創建並執行 HINCRBY 命令。

如果指定的字段不存在，那麼在執行命令前，字段的值被初始化為 0 。

對一個儲存字符串值的字段執行 HINCRBY 命令將返回false。

本操作的值被限制在 64 位(bit)有符號數字表示之內。

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

類似與hIncrBy，只不過增量是浮點型。

## **hMSet**

同時將多個 field-value (字段-值)對設置到哈希表中。

此命令會覆蓋哈希表中已存在的字段。

如果哈希表不存在，會創建一個空哈希表，並執行 HMSET 操作。

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

以關聯陣列的形式返回哈希表中一個或多個給定字段的值。

如果指定的字段不存在於哈希表，那麼對應的字段將是null值 。如果key不是hash則返回false。

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
輸出
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```

## **blPop, brPop**

移出並獲取列表的第一個元素/最後一個元素， 如果列表沒有元素會阻塞列表直到等待超時或發現可彈出元素為止。

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

## **bRPopLPush**

從列表中取出最後一個元素，並插入到另外一個列表的頭部； 如果列表沒有元素會阻塞列表直到等待超時或發現可彈出元素為止。如果超時則返回null。

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

## **lIndex**

通過索引獲取列表中的元素。你也可以使用負數下標，以 -1 表示列表的最後一個元素， -2 表示列表的倒數第二個元素，以此類推。

 如果指定索引值不在列表的區間範圍內，返回 null。如果對應的key不是列表類型，則返回false。

```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

在列表的元素前或者後插入元素。當指定元素不存在於列表中時，不執行任何操作。

當列表不存在時，被視為空列表，不執行任何操作。如果 key 不是列表類型則返回false。

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

## **lPop**

移除並返回列表的第一個元素。

當列表 key 不存在時，返回 null 。

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```

## **lPush**

一個或多個值插入到列表頭部。 如果 key 不存在，一個空列表會被創建並執行 LPUSH 操作。 當 key 存在但不是列表類型時返回false。

**注意：**在Redis 2.4版本以前的 LPUSH 命令，都只接受單個 value 值。

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lPushx**

將一個值插入到已存在的列表頭部，列表不存在時操作無效，返回0。如果key不是列表類型則返回false。

返回值為lPushx 命令執行之後，列表的長度。

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

## **lRange**

返回列表中指定區間內的元素，區間以偏移量 START 和 END 指定。 其中 0 表示列表的第一個元素， 1 表示列表的第二個元素，以此類推。 你也可以使用負數下標，以 -1 表示列表的最後一個元素， -2 表示列表的倒數第二個元素，以此類推。

返回一個包含指定區間內的元素陣列。如果key不是列表類型則返回false。

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

根據參數 COUNT 的值，移除列表中與參數 VALUE 相等的元素。

COUNT 的值可以是以下幾種：

*   count > 0 : 從表頭開始向表尾搜索，移除與 VALUE 相等的元素，數量為 COUNT 。
*   count < 0 : 從表尾開始向表頭搜索，移除與 VALUE 相等的元素，數量為 COUNT 的絕對值。
*   count = 0 : 移除表中所有與 VALUE 相等的值。

返回被移除元素的數量。 列表不存在時返回 0。不是列表時返回false。

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

通過索引來設置元素的值。

成功時返回true，當索引參數超出範圍，或對一個空列表進行 LSET 時返回false。

```php
$redis->lSet('key1', 0, 'X');
```
## **lTrim**

進行列表修剪(trim)，即使列表只保留指定區間內的元素，不在指定區間之內的元素都將被刪除。

下標 0表示列表的第一個元素，以1表示列表的第二個元素，以此類推。 你也可以使用負數下標，以-1表示列表的最後一個元素，-2表示列表的倒數第二個元素，以此類推。

成功時返回true，失敗時返回false。

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

## **rPop**

用於移除列表的最後一個元素，返回值為移除的元素。

當列表不存在時，返回 null 。

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush**

移除列表的最後一個元素，並將該元素添加到另一個列表並返回。

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

## **rPush**

將一個或多個值插入到列表的尾部(最右邊)，並返回插入後的列表的長度。

如果列表不存在，一個空列表會被創建並執行 RPUSH 操作。 當列表存在但不是列表類型時返回false。

**注意：**在 Redis 2.4 版本以前的 RPUSH 命令，都只接受單個 value 值。

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX**

將一個值插入到已存在的列表尾部(最右邊)並返回列表的長度。如果列表不存在，操作無效，返回0。 當列表存在但不是列表類型時返回false。

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen**

返回列表的長度。 如果列表 key 不存在，則 key 被解釋為一個空列表，返回 0 。 如果 key不是列表類型，返回false。

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lLen('key1', function ($r) {
    var_dump($r); // 3
});
```

## **sAdd**

將一個或多個成員元素加入到集合中，已經存在於集合的成員元素將被忽略。

假如集合 key 不存在，則創建一個只包含添加的元素作成員的集合。

當集合 key 不是集合類型時返回false。

**注意：**在 Redis2.4 版本以前，SADD 只接受單個成員值。

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

## **sCard**

返回集合中元素的數量。當集合 key 不存在時返回 0 。

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

## **sDiff**

返回第一個集合與其他集合之間的差異，也可以認為說第一個集合中獨有的元素。不存在的集合 key 將視為空集。

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

## **sDiffStore**

給定集合之間的差集存儲在指定的集合中。如果指定的集合 key 已存在，則會被覆蓋。

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

## **sInter**

返回給定所有給定集合的交集。不存在的集合 key 被視為空集。當給定集合當中有一個空集時，結果也為空集。

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

## **sInterStore**

將給定集合之間的交集存儲在指定的集合中並返回存儲交集的集合的元素數量。如果指定的集合已經存在，則將其覆蓋。

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

## **sIsMember**

判斷成員元素是否是集合的成員。

如果成員元素是集合的成員返回 1 。 如果成員元素不是集合的成員，或 key 不存在返回 0 。如果 key對應的不是集合類型，則返回false。

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## **sMembers**

返回集合中的所有的成員。 不存在的集合 key 被視為空集合。

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## **sMove**

將指定成員 member 元素從source集合移動到destination集合。

SMOVE 是原子性操作。

如果source集合不存在或不包含指定的member元素，則SMOVE命令不執行任何操作，僅返回0 。否則，member元素從source集合中被移除，並添加到destination集合中去。

當destination集合已經包含member元素時，SMOVE命令只是簡單地將source集合中的member元素刪除。

當source或destination不是集合類型時返回false。

```php
$redis->sMove('key1', 'key2', 'member13');
```

## **sPop**

移除集合中的指定 key 的一個或多個隨機元素，移除後會返回移除的元素。

當集合不存在或是空集時，返回null 。

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

## **sRandMember**

Redis Srandmember 命令用於返回集合中的一個隨機元素。

從Redis 2.6版本開始，Srandmember命令接受可選的count參數：

*  如果count為正數，且小於集合基數，那麼命令返回一個包含count個元素的數組，數組中的元素各不相同。如果count大於等於集合基數，那麼返回整個集合。
*  如果count為負數，那么命令返回一個數組，數組中的元素可能會重複出現多次，而數組的長度為count的絕對值。

該操作和SPOP相似，但SPOP將隨機元素從集合中移除並返回，而Srandmember則僅僅返回隨機元素，而不對集合進行任何改動。

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
## **sRem**

移除集合中的一個或多個成員元素，不存在的成員元素會被忽略。

返回被成功移除的元素的數量，不包括被忽略的元素。

當 key 不是集合類型返回false。

在 Redis 2.4 版本以前， SREM 只接受單個成員值。

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

命令返回給定集合的並集。不存在的集合 key 被視為空集。

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

將給定集合的並集存儲在指定的集合 destination 中並返回元素數量。如果 destination 已經存在，則將其覆蓋。

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
