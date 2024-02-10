# 웹맨 레디스

## 소개

웍맨/레디스는 Workerman을 기반으로 한 비동기 레디스 컴포넌트입니다.

> **주의**
> 이 프로젝트의 주요 목적은 레디스 비동기 구독(subscribe, pSubscribe)을 구현하는 것입니다. 
> 레디스는 충분히 빠르기 때문에 pSubscribe subscribe 비동기 구독이 필요하지 않은 경우에는 이 비동기 클라이언트를 사용할 필요가 없으며, 레디스 확장을 사용하는 것이 성능면에서 더 나을 수 있습니다.

## 설치:
```composer require workerman/redis```

## 콜백 방법

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

## 코루틴 방법

> **주의**
> 코루틴 방법은 workerman>=5.0, workerman/redis>=2.0.0 이상 및 composer require revolt/event-loop ^1.0.0 설치가 필요합니다.

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

콜백 함수를 설정하지 않으면 클라이언트는 비동기 요청 결과를 동기적인 방식으로 반환하며, 요청 프로세스는 현재 프로세스를 차단하지 않으므로 병렬로 요청을 처리할 수 있습니다.

> **주의**
> psubscribe subscribe는 코루틴 방법을 지원하지 않습니다.

# 문서
**설명**

**콜백 방식에서 콜백 함수는 일반적으로 2개의 매개변수($result, $redis)를 가지며, `$result`는 결과이고, `$redis`는 레디스 인스턴스입니다. 예:**
```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// 콜백 함수 설정하여 set 호출 결과 확인
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// 콜백 함수는 선택적 매개변수이며, 여기서는 콜백 함수를 생략했습니다.
$redis->set('key1', 'value1');
// 콜백 함수는 중첩될 수 있습니다
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **연결**
```php
use Workerman\Redis\Client;
// 콜백 생략
$redis = new Client('redis://127.0.0.1:6379');
// 콜백 포함
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // 연결 제한시간 10초로 설정, 기본값은 5초
], function ($success, $redis) {
    // 연결 결과 콜백
    if (!$success) echo $redis->error();
});
```

## **auth**
```php
// 비밀번호 검증
$redis->auth('password', function ($result) {
    
});
// 사용자 이름과 비밀번호 검증
$redis->auth('username', 'password', function ($result) {

});
```

## **pSubscribe**

주어진 패턴과 일치하는 하나 이상의 채널을 구독합니다.
각 패턴은 \*을 일치기호로 사용하며, 예를 들어 it\*은 it로 시작하는 모든 채널( it.news, it.blog, it.tweets 등)을 일치합니다. 또한 news.\*은 news로 시작하는 모든 채널( news.it, news.global.today 등)을 일치시킵니다.

주의: pSubscribe 콜백 함수에는 4개의 매개변수($pattern, $channel, $message, $redis)가 있습니다.

한번 $redis 인스턴스가 pSubscribe 또는 subscribe 인터페이스를 호출한 후에는 현재 인스턴스에서 다른 메소드를 호출해도 무시됩니다.

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

특정 채널의 정보를 구독합니다.

주의: subscribe 콜백 함수에는 3개의 매개변수($channel, $message, $redis)가 있습니다.

한번 $redis 인스턴스가 pSubscribe 또는 subscribe 인터페이스를 호출한 후에는 현재 인스턴스에서 다른 메소드를 호출해도 무시됩니다.

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

지정된 채널로 정보를 발송합니다.

정보를 수신한 구독자의 수를 반환합니다.

```php
$redis2->publish('news', 'news content');
```


## **select**
```php
// 콜백 생략
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // select 매개변수는 반드시 숫자이어야 하므로 여기서 $result는 false입니다.
    var_dump($result, $redis->error());
});
```

## **get**

지정된 키의 값을 가져오는 명령어입니다. 키가 존재하지 않으면 NULL을 반환합니다. 또한, 키에 저장된 값이 문자열 형식이 아니거나 존재하지 않는 경우 false를 반환합니다.
```php
$redis->get('key', function($result) {
     // 키가 존재하지 않으면 NULL을 반환하며, 오류가 발생하면 false를 반환합니다.
    var_dump($result);
});
```

## **set**

지정된 키의 값을 설정하는 명령어입니다. 이미 키에 다른 값이 저장되어 있다면 이전 값을 무시하고 덮어씁니다.
```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// 세 번째 매개변수로 만료 시간을 전달할 수 있으며, 만료 시간은 10초 뒤로 설정됩니다.
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **setEx, pSetEx**

지정된 키의 값과 만료 시간을 설정하는 명령어입니다. 이미 키가 존재한다면 pSetEx 명령어는 이전 값을 대체합니다.

```php
// 두번째 매개변수로 만료 시간을 전달하며, 시간 단위는 초입니다.
$redis->setEx('key', 3600, 'value'); 
// pSetEx는 시간 단위가 밀리초(millisecond)입니다.
$redis->pSetEx('key', 3600, 'value'); 
```

## **del**

존재하는 키를 삭제하는 명령어로, 결과는 삭제된 키의 수를 나타냅니다(존재하지 않는 키는 카운트되지 않습니다).
```php
// 하나의 키를 삭제합니다.
$redis->del('key');
// 여러 개의 키를 삭제합니다.
$redis->del(['key', 'key1', 'key2']);
````

## **setNx**

(**SET**if**N**ot e**X**ists) 명령어는 지정된 키가 없을 때 해당 키에 값을 설정하는 명령어입니다.
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

지정된 키가 존재하는지 확인하는 명령어로, 결과는 키의 수를 나타냅니다.
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
  
지정된 키에 저장된 숫자 값을 1씩 증가시키거나 지정된 값을 증가시키는 명령어입니다. 키가 존재하지 않는 경우에는 0으로 초기화한 후 증가 작업을 수행합니다.
값의 유형이 잘못되었거나 문자열 형태로 표현할 수 없는 경우 false를 반환합니다.
성공하면 증가된 수치를 반환합니다.
```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **incrByFloat**

지정된 키에 저장된 값에 지정된 부동 소수점 증분 값을 더하는 명령어입니다.
키가 존재하지 않는 경우 INCRBYFLOAT 명령어는 키의 값을 0으로 설정한 후 증가 작업을 수행합니다.
값의 유형이 잘못되었거나 문자열 형태로 표현할 수 없는 경우 false를 반환합니다.
성공하면 증가된 수치를 반환합니다.
```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## **decr, decrBy**

지정된 키에 저장된 값에서 1씩 뺄 뿐이거나 지정된 값을 감소시키는 명령어입니다.
키가 존재하지 않는 경우에는 0으로 초기화한 후 감소 작업을 수행합니다.
값의 유형이 잘못되었거나 문자열 형태로 표현할 수 없는 경우 false를 반환합니다.
성공하면 감소된 수치를 반환합니다.
```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

주어진 키의 값을 반환하는 명령어입니다. 주어진 키 중 하나라도 존재하지 않으면 해당 키는 NULL을 반환합니다.
```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
## **getSet**

지정된 키의 값을 설정하고 해당 키의 이전 값을 반환합니다.

```php
$redis->set('x', '42');
$redis->getSet('x', 'lol', function ($result) {
    var_dump($result); // '42'
});
$redis->get('x', function ($result) {
    var_dump($result); // 'lol'
});
```

## **randomKey**

현재 데이터베이스에서 무작위로 한 개의 키를 반환합니다.
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    });
});
```

## **move**

현재 데이터베이스의 키를 지정된 데이터베이스 db로 이동시킵니다.
```php
$redis->select(0);	// DB 0로 전환
$redis->set('x', '42');	// x에 42를 작성
$redis->move('x', 1, function ($result) { 	// DB 1로 이동
    var_dump($result); // 1
});  
$redis->select(1);	// DB 1로 전환
$redis->get('x', function ($result) {
    var_dump($result); // '42'
});
```

## **rename**

키의 이름을 수정하고, 키가 없는 경우 false를 반환합니다.
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
});
```

## **renameNx**

새 키가 존재하지 않을 때만 키의 이름을 수정합니다.
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
});
```

## **expire**

키의 만료 시간을 설정하여 만료되면 더 이상 사용할 수 없게 합니다. 초로 표시됩니다. 성공 시 1을 반환하고, 키가 없을 경우 0을 반환하며, 오류가 발생하면 false를 반환합니다.
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

주어진 패턴과 일치하는 모든 키를 찾습니다.
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
});
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
});
```

## **type**

키에 저장된 값의 유형을 반환합니다. 결과는 문자열이며, string set list zset hash none 중 하나이며, none은 키가 존재하지 않음을 나타냅니다.
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
});
```

## **append**

키가 이미 있고 문자열이면, APPEND 명령은 값을 기존 값의 끝에 추가하고 문자열의 길이를 반환합니다.

키가 없으면 간단히 지정된 키 값을 값으로 설정하고 SET key value를 실행한 것과 같은 방식으로 문자열의 길이를 반환합니다.

키가 존재하지만 문자열이 아닌 경우에는 false를 반환합니다.
```php
$redis->set('key', 'value1');
$redis->append('key', 'value2', function ($result) {
    var_dump($result); // 12
});
$redis->get('key', function ($result) {
    var_dump($result); // 'value1value2'
});
```

## **getRange**

지정된 키에 저장된 문자열의 하위 문자열을 가져옵니다. 문자열의 잘라내기 범위는 시작과 끝 오프셋으로 결정됩니다(시작과 끝을 포함합니다). 키가 존재하지 않으면 빈 문자열을 반환하며, 키의 유형이 문자열이 아닐 경우 false를 반환합니다.
```php
$redis->set('key', 'string value');
$redis->getRange('key', 0, 5, function ($result) {
    var_dump($result); // 'string'
});
$redis->getRange('key', -5, -1 , function ($result) {
    var_dump($result); // 'value'
});
```

## **setRange**

지정된 문자열로 지정된 키에 저장된 문자열 값을 덮어씁니다. 시작 오프셋부터 덮어씁니다. 키가 없으면 해당 문자열으로 키를 설정합니다. 키의 유형이 문자열이 아닐 경우 false를 반환합니다.

결과는 변경된 문자열의 길이입니다.
```php
$redis->set('key', 'Hello world');
$redis->setRange('key', 6, "redis", function ($result) {
    var_dump($result); // 11
});
$redis->get('key', function ($result) {
    var_dump($result); // 'Hello redis'
});
```

## **strLen**

지정된 키에 저장된 문자열 값의 길이를 얻습니다. 키가 문자열 값이 아닌 경우 false를 반환합니다.
```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
});
```

## **getBit**

지정된 키에 저장된 문자열 값에서 지정된 오프셋의 비트를 가져옵니다.
```php
$redis->set('key', "\x7f"); // 이 값은 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
});
```

## **setBit**

지정된 키에 저장된 문자열 값에서 지정된 오프셋의 비트를 설정하거나 지웁니다. 결과는 수정 전의 값인 0 또는 1입니다.
```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2a = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
});
```

## **bitOp**

여러 키(문자열 값 포함) 사이에서 비트 연산을 수행하고 결과를 대상 키에 저장합니다.

BITOP 명령은 AND, OR, XOR 및 NOT 네 가지 비트 연산을 지원합니다.

결과는 대상 키에 저장된 문자열의 크기로, 입력 문자열 중 가장 긴 것과 같습니다.
```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
});
```

## **bitCount**

문자열의 설정된 비트 수(인구 수)를 계산합니다.

기본적으로 문자열에 포함된 모든 바이트를 검사합니다. 추가 매개변수 start와 end 간격에서만 계산 작업을 지정할 수 있습니다.

GETRANGE 명령과 유사하게 시작과 끝은 문자열의 끝에서부터 색인화하기 위해 음수 값을 포함하여 지정할 수 있습니다. 여기서 -1은 마지막 바이트이고, -2는 두 번째로 마지막 바이트입니다.

결과는 문자열에서 값이 1인 비트 수입니다.

존재하지 않는 키는 빈 문자열로 취급되므로 해당 명령은 0을 반환합니다.
```php
$redis->set('key', 'hello');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
});
$redis->bitCount( 'key', function ($result) {
    var_dump($result); //21
});
```

## **sort**

sort 명령을 사용하여 list, set 및 sorted set의 요소를 정렬할 수 있습니다.

원형: `sort($key, $options, $callback);`

여기서 options는 다음과 같은 선택적인 키와 값입니다.
```
$options = [
     'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // 또는 패턴 배열
    'sort' => 'asc', // 또는 'desc'
    'alpha' => true,
    'store' => 'external-key'
];
```
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

키의 남은 만료 시간을 초/밀리초 단위로 반환합니다.

키에 만료가 없으면 -1을 반환합니다. 키가 없으면 -2를 반환합니다.
```php
$redis->set('key', 'value', 10);
// 초 단위
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// 밀리초 단위
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// 키가 없음
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```

## **persist**

지정된 키의 만료 시간을 제거하여 키가 영원히 만료되지 않도록 합니다.

성공적으로 만료 시간이 제거되면 1을 반환하고, 키가 없거나 만료 시간이 없으면 0을 반환하며, 오류가 발생하면 false를 반환합니다.
```php
$redis->persist('key');
```

## **mSet, mSetNx**

단일 명령으로 여러 키-값 쌍을 설정합니다. mSetNx는 모든 키가 설정된 경우에만 1을 반환합니다.

성공하면 1을 반환하고, 실패하면 0을 반환하며, 오류가 발생하면 false를 반환합니다.
```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

해시 테이블의 필드에 값을 할당합니다.

필드가 해시 테이블에 새로운 필드인 경우 및 값을 성공적으로 설정한 경우 1을 반환합니다. 필드가 이미 해시 테이블에 있고 이전 값을 새 값으로 덮어쓴 경우 0을 반환합니다.
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

해시 테이블에 존재하지 않는 필드에 값을 설정합니다.

해시 테이블이 없는 경우 새 해시 테이블이 생성되어 HSET 작업이 수행됩니다.

필드가 이미 해시 테이블에 있으면 작업은 무효화됩니다.

키가 없으면 새 해시 테이블이 생성되고 HSETNX 명령이 실행됩니다.
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

지정된 해시 테이블의 필드의 값을 반환합니다.

지정된 필드 또는 키가 없는 경우 null을 반환합니다.
```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```
## **hLen**

해시 테이블에서 필드 수를 가져오는 데 사용됩니다.

키가 없는 경우 0을 반환합니다.

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

해시 테이블에서 지정된 하나 이상의 필드를 삭제하는 데 사용됩니다. 존재하지 않는 필드는 무시됩니다.

성공적으로 삭제된 필드의 수를 반환하며 무시된 필드는 포함되지 않습니다. 키가 해시 테이블이 아닌 경우 false가 반환됩니다.

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

해시 테이블의 모든 필드를 배열 형식으로 가져옵니다.

키가 없는 경우 빈 배열을 반환하며 해당 키가 해시 테이블이 아닌 경우 false가 반환됩니다.

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

해시 테이블의 모든 필드 값들을 배열 형식으로 반환합니다.

키가 없는 경우 빈 배열을 반환하며 해당 키가 해시 테이블이 아닌 경우 false가 반환됩니다.

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

해시 테이블의 모든 필드와 값을 연관 배열 형식으로 반환합니다.

키가 없는 경우 빈 배열을 반환하며 해당 키가 해시 테이블이 아닌 경우 false가 반환됩니다.

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
반환
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```
## **hExists**

해시 테이블에서 지정된 필드가 존재하는지 확인합니다. 필드가 존재하면 1을 반환하며, 필드나 키가 존재하지 않으면 0을 반환합니다. 오류가 발생하면 false를 반환합니다.

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## **hIncrBy**

해시 테이블의 필드 값에 지정된 증분 값을 더하는 데 사용됩니다.

증분은 음수일 수도 있으며, 지정된 필드가 없는 경우 HINCRBY 명령을 실행하기 전에 해당 필드의 값은 0으로 초기화됩니다.

문자열 값이 저장된 필드에 HINCRBY 명령을 실행하면 false가 반환됩니다.

이 작업의 값은 64비트 부호 있는 숫자 표현 내에 제한됩니다.

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

hIncrBy와 유사하지만 증분 값이 부동 소수점입니다.

## **hMSet**

하나 이상의 필드-값 쌍을 해시 테이블에 설정합니다. 이 명령은 기존의 필드를 덮어씁니다. 해시 테이블이 없는 경우 빈 해시 테이블을 생성하고 HMSET 작업을 실행합니다.

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

지정된 필드의 값을 연관 배열 형식으로 반환합니다. 해시 테이블에 지정된 필드가 없으면 해당 필드는 null 값으로 반환됩니다. 키가 해시 테이블이 아닌 경우 false가 반환됩니다.

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
출력
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```
## **blPop, brPop**

리스트의 첫 번째 요소/마지막 요소를 제거하고 가져옵니다. 리스트에 요소가 없는 경우 요소가 팝될 때까지 대기합니다.

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

리스트에서 마지막 요소를 가져와 다른 리스트의 처음에 삽입합니다. 리스트에 요소가 없는 경우 요소가 팝될 때까지 대기합니다. 타임아웃이 발생하면 null이 반환됩니다.

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

리스트에서 인덱스로 요소를 가져옵니다. 음수 인덱스를 사용하여 -1은 리스트의 마지막 요소를 나타내며, -2는 뒤에서 두번째 요소를 나타냅니다.

지정된 인덱스 값이 리스트의 범위를 벗어나면 null을 반환하며, 해당 키가 리스트 타입이 아닌 경우 false를 반환합니다.

```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

리스트의 요소 앞이나 뒤에 요소를 삽입합니다. 지정된 요소가 리스트에 없는 경우 아무 작업도 수행하지 않습니다. 리스트가 없으면 빈 리스트로 간주되어 아무 작업도 수행하지 않습니다. 키가 리스트 타입이 아닌 경우 false를 반환합니다.

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

리스트의 첫 번째 요소를 제거하고 반환합니다.

리스트 키가 없는 경우 null을 반환합니다.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```

## **lPush**

하나 이상의 값을 리스트의 맨 앞에 삽입합니다. 키가 없는 경우 빈 리스트를 생성하고 LPUSH 작업을 실행합니다. 키가 리스트 타입이 아닌 경우 false를 반환합니다.

**참고:** Redis 2.4 버전 이전의 LPUSH 명령은 단일 값만 수용합니다.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lPushx**

기존 리스트의 맨 앞에 값 하나를 삽입합니다. 리스트가 없는 경우 작업은 무효화되고 0을 반환합니다. 키가 리스트 타입이 아닌 경우 false를 반환합니다.

lPushx 명령 실행 후 리스트의 길이 값이 반환됩니다.

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

지정된 범위 내의 요소를 리스트에서 반환합니다. 0은 리스트의 첫 번째 요소를 나타내며, 1은 두 번째 요소를 나타냅니다. 음수 인덱스를 사용하여 -1은 리스트의 마지막 요소를 나타내며, -2는 뒤에서 두번째 요소를 나타냅니다.

지정된 범위의 요소 배열을 반환합니다. 키가 리스트 타입이 아닌 경우 false를 반환합니다.

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

COUNT 매개변수의 값에 따라 리스트에서 값이 VALUE와 동일한 요소를 제거합니다.

COUNT 값은 다음과 같은 여러 가지 경우의 수를 가질 수 있습니다:

* count > 0 : 리스트의 맨 앞부터 리스트 끝까지 VALUE와 동일한 요소를 COUNT만큼 제거합니다.
* count < 0 : 리스트의 맨 끝부터 리스트 앞까지 VALUE와 동일한 요소를 COUNT의 절대값만큼 제거합니다.
* count = 0 : 리스트에서 VALUE와 동일한 값을 모두 제거합니다.

제거된 요소의 수를 반환합니다. 리스트가 없는 경우 0을 반환합니다. 리스트 타입이 아닌 경우 false를 반환합니다.

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

인덱스를 사용하여 요소의 값을 설정합니다.

성공시 true를 반환하며 인덱스 매개 변수가 범위를 벗어나거나 빈 리스트에 LSET을 수행하는 경우 false를 반환합니다.

```php
$redis->lSet('key1', 0, 'X');
```
## **lTrim**

리스트를 잘라내는 것은 리스트에 지정된 범위 내의 요소만 유지하고 범위 이외의 요소는 모두 삭제하는 것을 말합니다.

인덱스 0은 리스트의 첫 번째 요소를 나타내며 1은 두 번째 요소를 나타내며 이와 같이 계속됩니다. 음수 인덱스를 사용할 수도 있습니다. -1은 리스트의 마지막 요소를 나타내고, -2는 끝에서 두 번째 요소를 나타내며, 이와 같이 계속됩니다.

성공하면 true를 반환하고 실패하면 false를 반환합니다.

```php
$redis->del('key1');

$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, 1, function ($r) {
    var_dump($r); // ['A', 'B']
});
$redis->lTrim('key1', 0, 1);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B']
});
```

## **rPop**

리스트에서 마지막 요소를 제거하고 제거된 요소를 반환합니다.

리스트가 존재하지 않을 때는 null을 반환합니다.

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush**

리스트의 마지막 요소를 제거하고 해당 요소를 다른 리스트에 추가하여 반환합니다.

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

하나 이상의 값을 리스트의 끝(가장 오른쪽)에 삽입하고 삽입 후 리스트의 길이를 반환합니다.

리스트가 존재하지 않으면 빈 리스트가 생성되고 RPUSH 작업이 실행됩니다. 리스트가 존재하지만 리스트 유형이 아닌 경우 false가 반환됩니다.

**참고:** Redis 2.4 버전 이전의 RPUSH 명령은 모두 단일 value 값을 받습니다.

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX**

기존 리스트의 끝(가장 오른쪽)에 값을 삽입하고 리스트의 길이를 반환합니다. 리스트가 없으면 작업은 무효화되고 0을 반환합니다. 리스트가 존재하지만 리스트 유형이 아닌 경우 false를 반환합니다.

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen**

리스트의 길이를 반환합니다. 키가 존재하지 않으면 키는 빈 리스트로 해석되어 0을 반환합니다. 키가 리스트 유형이 아니면 false를 반환합니다.

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

하나 이상의 멤버 요소를 집합에 추가하고 이미 집합에 존재하는 멤버 요소는 무시합니다.

집합 key가 존재하지 않는 경우 해당 멤버만 포함하는 집합이 생성됩니다.

집합 key가 집합 유형이 아닌 경우 false를 반환합니다.

**참고:** Redis 2.4 버전 이전에는 SADD가 단일 멤버 값을 받습니다.

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

집합의 요소 수를 반환합니다. 집합 key가 존재하지 않는 경우 0을 반환합니다.

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

첫 번째 집합과 다른 집합들 사이의 차이를 반환하며, 첫 번째 집합에만 있는 요소로 간주됩니다. 존재하지 않는 집합 key는 빈 집합으로 간주됩니다.

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

지정된 집합 사이의 차집합을 지정된 집합에 저장합니다. 지정된 집합 key가 이미 존재하는 경우 덮어씁니다.

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

지정된 모든 집합의 교집합을 반환합니다. 존재하지 않는 집합 key는 빈 집합으로 간주되며, 공집합일 경우 결과도 공집합입니다.

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

지정된 집합 간의 교집합을 지정된 집합에 저장하고 저장된 교집합 집합의 요소 수를 반환합니다. 지정된 집합이 이미 존재하는 경우 덮어쓰기를 합니다.

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

멤버 요소가 집합의 멤버인지 여부를 판단합니다.

멤버 요소가 집합의 멤버이면 1을 반환하고, 멤버 요소가 집합의 멤버가 아니거나 키가 존재하지 않으면 0을 반환합니다. 또한 키의 형식이 집합이 아닌 경우 false를 반환합니다.

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## **sMembers**

집합의 모든 멤버를 반환합니다. 키가 존재하지 않으면 빈 집합으로 간주됩니다.

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## **sMove**

지정된 멤버 요소를 출발 집합에서 대상 집합으로 이동합니다.

SMOVE는 원자성 연산입니다.

출발 집합이 존재하지 않거나 지정된 멤버가 포함되지 않은 경우 SMOVE 명령은 어떠한 작업도 수행하지 않고 0만 반환합니다. 그렇지 않으면 멤버 요소가 출발 집합에서 제거되고 대상 집합에 추가됩니다.

출발 또는 대상이 집합 유형이 아닐 경우 false를 반환합니다.

```php
$redis->sMove('key1', 'key2', 'member13');
```

## **sPop**

집합에서 지정된 키에 있는 하나 이상의 임의의 요소를 제거하고 제거한 요소를 반환합니다.

집합이 존재하지 않거나 비어있는 경우 null을 반환합니다.

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

Redis Srandmember 명령어는 집합에서 임의의 요소를 반환합니다.

Redis 2.6 버전부터 Srandmember 명령에 선택적으로 count 매개변수를 받을 수 있습니다:

* count가 양수이고 집합의 크기보다 작은 경우, 명령은 고유한 count 요소를 포함하는 배열을 반환합니다. count가 집합 크기보다 큰 경우 전체 집합을 반환합니다.
* count가 음수인 경우, 명령은 count의 절대값의 길이를 갖는 배열을 반환하며, 배열의 요소는 여러 번 나타날 수 있습니다.

이 작업은 SPOP과 유사하지만, SPOP은 집합에서 요소를 제거하고 반환하는 반면, Srandmember는 집합을 변경하지 않고 단순히 반환합니다.

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
## **sRem (삭제)**

집합에서 하나 이상의 멤버 요소를 제거하고, 존재하지 않는 멤버 요소는 무시됩니다.

삭제된 요소의 수를 반환하며, 무시된 요소는 포함되지 않습니다.

키가 집합 유형이 아닌 경우에는 false를 반환합니다.

Redis 2.4 버전 이전에는 sRem은 단일 멤버 값을만 허용했습니다.

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion (합집합)**

명령어는 주어진 집합의 합집합을 반환합니다. 존재하지 않는 집합 키는 빈 집합으로 간주됩니다.

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore (합집합 저장)**

주어진 집합의 합집합을 지정된 집합 대상에 저장하고 요소 수를 반환합니다. 대상이 이미 존재하는 경우 덮어쓸 것입니다.

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
