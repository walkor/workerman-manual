# workerman-redis

## イントロダクション

workeman/redisは、workermanをベースとした非同期Redisコンポーネントです。

> **注意**
> このプロジェクトの主な目的はRedisの非同期サブスクライブ（subscribe、pSubscribe）を実装することです。
> Redisは非常に高速ですので、psubscribe subscribeの非同期サブスクライブが必要ない限り、この非同期クライアントを使用する必要はありません。Redis拡張機能を使用した方が性能が向上します。

## インストール
```shell
composer require workerman/redis
```

## コールバック方式

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

## コルーチンの使用法

> **注意**
> コルーチンの使用法にはworkerman>=5.0、workerman/redis>=2.0.0が必要で、 composer require revolt/event-loop ^1.0.0をインストールする必要があります。

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

コールバック関数を設定しない場合、クライアントは同期的に非同期リクエストの結果を返し、リクエストプロセスは現在のプロセスをブロックしません。つまり、並行してリクエストを処理できます。

> **注意**
> psubscribe subscribeはコルーチン使用法をサポートしていません。

# ドキュメント
**説明**

**コールバック方式では、コールバック関数には通常2つのパラメータ（$result、$redis）があり、`$result`は結果、`$redis`はRedisのインスタンスです。例：**
```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// コールバック関数を設定してsetの呼び出し結果を判断します
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// コールバック関数はオプションのパラメータです。ここでは省略しました。
$redis->set('key1', 'value1');
// コールバック関数をネストさせることができます
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **接続**
```php
use Workerman\Redis\Client;
// コールバックを省略
$redis = new Client('redis://127.0.0.1:6379');
// コールバックを伴う
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // 接続タイムアウトを10秒に設定、デフォルトは5秒
], function ($success, $redis) {
    // 接続結果のコールバック
    if (!$success) echo $redis->error();
});
```

## **auth**
```php
// パスワード認証
$redis->auth('password', function ($result) {
    
});
// ユーザー名とパスワードの認証
$redis->auth('username', 'password', function ($result) {

});
```

## **pSubscribe**

特定のパターンに一致する一つ以上のチャネルをサブスクライブするために使用されます。

各パターンは、\*をワイルドカードとして使用し、例えば it\* は itで始まるすべてのチャネル（it.news、it.blog、it.tweets など）に一致します。 news.\* は news.で始まるすべてのチャネル（news.it、news.global.today など）に一致します。

注意：psubscribeのコールバック関数には4つのパラメータ（$pattern, $channel, $message, $redis）があります。

$redisインスタンスがpsubscribeまたはsubscribeメソッドを呼び出した後、現在のインスタンスで他のメソッドを呼び出しても無視されます。

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

指定した1つまたは複数のチャネルの情報をサブスクライブするために使用されます。

注意：subscribeのコールバック関数には3つのパラメータ（$channel, $message, $redis）があります。

$redisインスタンスがpsubscribeまたはsubscribeメソッドを呼び出した後、現在のインスタンスで他のメソッドを呼び出しても無視されます。

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

指定されたチャネルに情報を送信するために使用されます。

情報を受け取ったサブスクライバーの数が返されます。

```php
$redis2->publish('news', 'news content');
```

## **select**
```php
// コールバックを省略
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // selectのパラメータは数字でなければならないため、ここでは$resultはfalseです
    var_dump($result, $redis->error());
});
```

## **get**

このコマンドは指定されたkeyの値を取得するために使用されます。keyが存在しない場合はNULLが返されます。keyの値が文字列型でない場合はfalseが返されます。
```php
$redis->get('key', function($result) {
    // keyが存在しない場合はNULLが返され、エラーが発生した場合はfalseが返されます
    var_dump($result);
});
```

## **set**

指定されたkeyの値を設定します。keyが既に他の値を格納している場合は、古い値を上書きし、タイプを無視します。
```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// 3番目のパラメータには有効期限を渡すことができます。10秒後に期限切れとなります
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **setEx, pSetEx**

指定されたkeyの値および有効期限を設定します。keyが存在する場合、SETEXコマンドは古い値を置き換えます。

```php
// 2番目のパラメータに有効期限を渡してください、単位は秒です
$redis->setEx('key', 3600, 'value');
// pSetExの単位はミリ秒です
$redis->pSetEx('key', 3600, 'value');
```

## **del**

すでに存在するキーを削除するために使用され、結果は削除されたキーの数量を示します（存在しないキーはカウントされません）。

```php
// 1つのキーを削除する
$redis->del('key');
// 複数のキーを削除する
$redis->del(['key', 'key1', 'key2']);
```

## **setNx**

（**SET**if**N**ot e**X**ists）命令は、指定したkeyが存在しない場合に、指定した値を設定します。
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

指定したkeyが存在するかどうかを確認するために使用されます。結果は存在するkeyの数を示します。
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

指定されたkeyに格納されている数値を増やします。keyが存在しない場合は、まずkeyの値が0に初期化され、次にincr/incrBy操作が実行されます。
値が間違ったタイプである、または文字列型の値が数字として表すことができない場合は、falseが返されます。
成功すると、増加後の数値が返されます。

```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **incrByFloat**

指定したkeyに格納されている値に指定された浮動小数点数を増やします。keyが存在しない場合は、まずkeyの値が0に初期化され、次にincrByFloat操作が実行されます。
値が間違ったタイプである、または文字列型の値が数字として表すことができない場合は、falseが返されます。
成功すると、増加後の数値が返されます。

```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## **decr, decrBy**

コマンドは、keyに格納されている値を1つ減算する/指定された減算値を減算します。
keyが存在しない場合、keyの値はまず0に初期化され、その後decr/decrBy操作が実行されます。
値に誤ったタイプが含まれている場合や、文字列タイプの値を数値で表現できない場合は、falseを返します。
成功した場合、減少後の数値が返されます。
```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

与えられたkeyの値をすべて（1つまたは複数）返します。与えられたkeyの中に、存在しないkeyがある場合、そのkeyはNULLを返します。
```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```

## **getSet**

指定したkeyの値を設定し、keyの古い値を返します。
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

現在のデータベースからランダムなkeyを返します。
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    }) ;
})
```

## **move**

現在のデータベースのkeyを指定されたデータベースdbに移動します。
```php
$redis->select(0);	// DB 0に切り替え
$redis->set('x', '42');	// xに42を書き込む
$redis->move('x', 1, function ($result) { 	// DB 1に移動
    var_dump($result); // 1
}) ;  
$redis->select(1);	// DB 1に切り替え
$redis->get('x', function ($result) {
    var_dump($result); // '42'
}) ;
```

## **rename**

keyの名前を変更します。keyが存在しない場合はfalseを返します。
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
}) ;
```

## **renameNx**

新しいkeyが存在しない場合にkeyの名前を変更します。
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
}) ;
```

## **expire**

keyの有効期限を設定し、有効期限が切れるとkeyは使用できなくなります。単位は秒です。成功すると1を返し、keyが存在しない場合は0を返し、エラーが発生した場合はfalseを返します。
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

指定したパターンに一致するすべてのkeyを検索します。
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
}) ;
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
}) ;
```

## **type**

keyに格納されている値のタイプを返します。結果は、文字列、set、list、zset、hash、noneのいずれかの文字列です。noneはkeyが存在しないことを示します。
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
}) ;
```

## **append**

keyが既に存在し、文字列である場合、APPENDコマンドはvalueを元の値の末尾に追加し、文字列の長さを返します。

keyが存在しない場合、APPENDは単純に指定されたkeyをvalueに設定し、SET key valueを実行したのと同じように、文字列の長さを返します。

keyが存在するが文字列ではない場合はfalseを返します。

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

指定したkeyに格納されている文字列の中から部分文字列を取得します。文字列の切り取り範囲は、startとendの2つのオフセットによって決まります（startとendを含む）。keyが存在しない場合、空の文字列を返します。keyが文字列タイプでない場合はfalseを返します。

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

指定した文字列で指定したkeyの文字列値を上書きします。上書きの位置はオフセットoffsetから始まります。keyが存在しない場合は、指定した文字列でkeyを設定します。keyが文字列でない場合はfalseを返します。

変更後の文字列の長さが返ります。

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

指定したkeyに格納されている文字列の長さを取得します。keyに文字列値が格納されていない場合は、falseが返ります。

```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
}) ; 
```

## **getBit**

keyに格納されている文字列値から、指定したオフセットのビット（bit）を取得します。

```php
$redis->set('key', "\x7f"); // これは0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **setBit**

keyに格納されている文字列値から、指定したオフセットのビット（bit）を設定またはクリアします。
返り値は変更前の値が0または1です。

```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2f = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **bitOp**

複数のキー（文字列値を含む）に対してビット単位の操作を行い、結果をターゲットキーに保存します。

BITOPコマンドは、AND、OR、XOR、NOTの4つのビット演算をサポートしています。

結果は、ターゲットキーに保存される文字列のサイズとなり、これは最長の入力文字列のサイズと同じです。

```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
}) ;
```

## **bitCount**

文字列内のセットビット数（ポピュレーションカウント）を計算します。

デフォルトでは、すべてのバイトをチェックします。追加の *start* と *end* の間隔を渡すことで、カウント操作を指定できます。

GETRANGEコマンドと同様に、startとendには負の値を含めることができ、文字列の末尾からバイトをインデックス付けするため、-1は最後のバイト、-2は最後から2番目のバイトなどとなります。

返り値は文字列内の値が1のビット数です。

存在しないキーは空文字列とみなされるため、このコマンドは0を返します。

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

sortコマンドは、リスト、セット、およびソート済みセットの要素をソートできます。

シグネチャ：sort($key, $options, $callback);

ここでoptionsは以下のオプションのキーと値です
~~~
$options = [
     'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // またはパターンの配列
    'sort' => 'asc', // または 'desc'
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

秒/ミリ秒単位でkeyの残りの有効期間を返します。

keyに有効期限がない場合は-1を返し、keyが存在しない場合は-2を返します。

```php
$redis->set('key', 'value', 10);
// 秒
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// ミリ秒
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// keyが存在しない場合
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```
## **persist**

指定されたキーの有効期限を削除し、キーを永続化します。

成功した場合は1を返し、キーが存在しないか有効期限が設定されていない場合は0を返します。エラーが発生した場合はfalseを返します。
```php
$redis->persist('key');
```

## **mSet, mSetNx**

複数のキーと値を一度に設定する原子コマンドです。mSetNxはすべてのキーが設定された場合にのみ1を返します。

成功した場合は1を返し、失敗した場合は0を返し、エラーが発生した場合はfalseを返します。
```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

ハッシュテーブル内のフィールドに値を設定します。

フィールドがハッシュテーブル内の新しいフィールドであり、かつ値が設定に成功した場合は1を返します。既存のフィールドが存在し、旧値が新しい値で上書きされた場合は0を返します。

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

ハッシュテーブル内の存在しないフィールドに値を設定します。

ハッシュテーブルが存在しない場合は新しいハッシュテーブルが作成され、HSET操作が行われます。

フィールドがすでにハッシュテーブル内に存在する場合は操作が無効になります。

キーが存在しない場合は新しいハッシュテーブルが作成され、HSETNXコマンドが実行されます。

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

指定したフィールドの値を返します。

指定したフィールドまたはキーが存在しない場合はnullを返します。

```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```

## **hLen**

ハッシュテーブル内のフィールドの数を取得します。

キーが存在しない場合は0を返します。

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

ハッシュテーブルから指定したフィールドを削除します。存在しないフィールドは無視されます。

削除されたフィールドの数を返しますが、無視されたフィールドは含まれません。キーがハッシュでない場合はfalseを返します。

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

ハッシュテーブル内のすべてのフィールドを配列形式で取得します。

キーが存在しない場合は空の配列を返し、ハッシュでない場合はfalseを返します。

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

ハッシュテーブル内のすべてのフィールドの値を配列形式で返します。

キーが存在しない場合は空の配列を返し、ハッシュでない場合はfalseを返します。

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

ハッシュテーブル内のすべてのフィールドと値を連想配列形式で返します。

キーが存在しない場合は空の配列を返し、ハッシュでない場合はfalseを返します。

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
返り値
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

## **hExists**

ハッシュテーブル内に指定したフィールドが存在するかどうかを確認します。フィールドが存在する場合は1を、フィールドまたはキーが存在しない場合は0を返し、エラーが発生した場合はfalseを返します。

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## **hIncrBy**

ハッシュテーブル内のフィールドの値に指定された増分値を加算します。

増分は負数にすることもでき、フィールドが存在しない場合はコマンドを実行する前にフィールドの値が0に初期化されます。

ストアされる値は64ビット符号付き数値で制限されます。

文字列値を持つフィールドにHINCRBYコマンドを実行するとfalseが返ります。

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

hIncrByと同様ですが、増分が浮動小数点数型です。

## **hMSet**

複数のフィールドと値を一度にハッシュテーブルに設定します。

このコマンドは既存のフィールドを上書きします。ハッシュテーブルが存在しない場合は空のハッシュテーブルが作成され、HMSET操作が実行されます。

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

指定したフィールドの値を連想配列でハッシュテーブルから取得します。

指定したフィールドがハッシュテーブルに存在しない場合、対応するフィールドはnull値になります。キーがハッシュでない場合はfalseを返します。

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
出力
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```

## **blPop, brPop**

リストの最初の要素/最後の要素を取り出して返します。リストが要素を持っていない場合は、要素が取り出されるまでリストがブロックされます。

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

リストから最後の要素を取り出し、別のリストの先頭に挿入します。リストが要素を持っていない場合は、要素が取り出されるまでリストがブロックされます。タイムアウトした場合はnullが返ります。

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

リスト内の要素をインデックスで取得します。負のインデックスを使用すると、-1はリストの最後の要素を、-2はリストの後ろから2番目の要素を表します。

指定されたインデックス値がリストの範囲外の場合はnullを返します。指定されたキーがリストでない場合はfalseを返します。

```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

リストの要素の前または後ろに新しい要素を挿入します。指定した要素がリスト内に存在しない場合は操作を行いません。

リストが存在しない場合は空のリストと見なされ、操作は行われません。キーがリストでない場合はfalseを返します。

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

リストの最初の要素を取り出して返します。

リストキーが存在しない場合はnullを返します。

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```
## **lPush**

要素を1つまたは複数をリストの先頭に挿入します。キーが存在しない場合、空のリストが作成され、LPUSH操作が実行されます。 ただし、キーが存在していてもリスト型でない場合はfalseが返されます。

**注意：**Redisの2.4バージョン以前のLPUSHコマンドは、単一の値のみを受け入れます。

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lPushx**

既存のリストの先頭に値を挿入し、リストが存在しない場合は操作が無効になり、0が返されます。また、キーがリスト型でない場合はfalseが返されます。

戻り値は、lPushxコマンドの実行後にリストの長さが返されます。

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

指定された範囲内の要素をリストから返します。範囲はオフセットのSTARTとENDで指定されます。 ここで、0はリストの最初の要素を示し、1はリストの2番目の要素を示し、これに続きます。 負数のインデックスも使用でき、-1はリストの最後の要素を示し、-2はリストの後ろから2番目の要素を示します。

指定された範囲内の要素を含む配列が返されます。キーがリスト型でない場合はfalseが返されます。

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

パラメータCOUNTの値に基づいて、リストからVALUEと等しい要素を削除します。

COUNTの値には、以下のいくつかの値が用いることができます：

*   count > 0 ： 先頭から末尾に向かってVALUEと等しい要素をCOUNT回削除します。
*   count < 0 ： 末尾から先頭に向かってVALUEと等しい要素をCOUNTの絶対値回削除します。
*   count = 0 ： VALUEと等しい値をリストからすべて削除します。

削除された要素の数が返されます。リストが存在しない場合は0が返されます。リストでない場合はfalseが返されます。

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

インデックスを使用して要素の値を設定します。

成功した場合はtrueが返されますが、インデックスパラメータが範囲外になった場合や空のリストに対してLSETを実行した場合はfalseが返されます。

```php
$redis->lSet('key1', 0, 'X');
```

## **lTrim**

リストをトリムし、指定された範囲内の要素のみが残るようにします。範囲内の要素以外は削除されます。

インデックス0はリストの最初の要素を、1はリストの2番目の要素を示し、このように続きます。 負数のインデックスも使用でき、-1はリストの最後の要素を示し、-2はリストの後ろから2番目の要素を示します。

成功した場合はtrueが返され、失敗した場合はfalseが返されます。

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

リストの最後の要素を削除し、削除された要素が返されます。

リストが存在しない場合、nullが返されます。

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush**

リストの最後の要素を削除し、その要素を別のリストに追加して返します。

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

1つまたは複数の値をリストの末尾（最右端）に挿入し、挿入後のリストの長さが返されます。

リストが存在しない場合、空のリストが作成され、RPUSH操作が実行されます。 ただし、リストが存在していてもリスト型でない場合はfalseが返されます。

**注意：**Redis 2.4バージョン以前のRPUSHコマンドは、単一の値のみを受け入れていました。

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX**

既存のリストの末尾（最右端）に値を挿入し、リストが存在しない場合は操作が無効になり、0が返されます。リストが存在していてもリスト型でない場合はfalseが返されます。

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen**

リストの長さを返します。リストキーが存在しない場合、そのキーは空のリストと解釈され、0が返されます。リストキーがリスト型でない場合はfalseが返されます。

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

1つまたは複数のメンバー要素をセットに追加し、セット内で既に存在する要素は無視されます。

セットキーが存在しない場合、追加された要素のみをメンバーとして含むセットが作成されます。

セットキーがセット型でない場合はfalseが返されます。

**注意：**Redis 2.4以前のバージョンでは、SADDは単一のメンバー値のみを受け入れていました。

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

セット内の要素の数を返します。セットキーが存在しない場合、0が返されます。

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

最初のセットと他のセットの間の差分が返されます。つまり、最初のセットにのみ存在する要素が含まれます。存在しないセットキーは空セットと見なされます。

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

指定されたセット間の差分を指定されたセットに保存します。指定されたセットキーが既に存在する場合は上書きされます。

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

指定されたすべてのセットの間の共通部分を返します。存在しないセットキーは空セットと見なされます。指定されたセットのどれか1つでも空セットの場合、結果も空セットになります。

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

指定された集合の間の共通部分を指定された集合に保存し、保存された共通部分の要素数を返します。指定された集合がすでに存在する場合は上書きします。

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

メンバー要素が集合のメンバーであるかどうかを判断します。

メンバー要素が集合のメンバーである場合は 1 を返します。メンバー要素が集合のメンバーではない場合、またはキーが存在しない場合は 0 を返します。キーが集合のタイプでない場合は false を返します。

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## **sMembers**

集合内のすべてのメンバーを返します。存在しない場合は空の集合と見なされます。

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## **sMove**

指定されたメンバー要素をソース集合から宛先集合に移動します。

SMOVE はアトミック操作です。

ソース集合が存在しないか、指定されたメンバー要素を含まない場合、SMOVE コマンドは何も実行せず、単に 0 を返します。それ以外の場合、メンバー要素はソース集合から削除され、宛先集合に追加されます。

宛先集合にすでにメンバー要素が含まれている場合、SMOVE コマンドは単純にソース集合からメンバー要素を削除します。

ソースまたは宛先が集合タイプでない場合は false を返します。

```php
$redis->sMove('key1', 'key2', 'member13');
```

## **sPop** 

指定したキーの集合から1つまたは複数のランダムな要素を削除し、削除した要素を返します。

集合が存在しないか空の場合は null を返します。

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

Redisの Srandmember コマンドは、集合から1つのランダムな要素を返します。

Redis 2.6 バージョンから、 Srandmember コマンドはオプションの count パラメータを受け入れます。

* count が正数で、集合のカーディナリティ未満の場合、count 個の要素を含む配列を返します。要素はすべて異なります。count が集合のカーディナリティ以上の場合、集合全体を返します。
* count が負数の場合、配列を返しますが、要素は count の絶対値の長さだけ繰り返し出現します。

この操作は SPOP と類似していますが、SPOP はランダムな要素を集合から削除して返すのに対し、Srandmember は集合を変更せずに単にランダムな要素を返します。

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

集合から1つまたは複数のメンバー要素を削除し、存在しないメンバー要素は無視されます。

削除された要素の数を返しますが、無視された要素は含まれません。キーが集合のタイプでない場合は false を返します。

Redis 2.4 バージョン以前では、SREM は単一のメンバー値のみを受け入れます。

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

指定された集合の和集合を返します。存在しない場合は空の集合と見なされます。

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

指定された集合の和集合を指定された宛先集合に保存し、要素の数を返します。宛先がすでに存在する場合は上書きされます。

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
