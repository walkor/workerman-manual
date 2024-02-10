# workerman-redis

## Введение

workeman/redis — это асинхронный компонент redis, основанный на workerman.

> **Примечание**
> Основная цель этого проекта - реализовать асинхронную подписку на redis (subscribe, pSubscribe). Поскольку redis достаточно быстрый, используйте этот асинхронный клиент только если требуется асинхронная подписка psubscribe или subscribe, иначе лучше для производительности использовать расширение redis.

## Установка:

```PHP
composer require workerman/redis
```

## Использование обратного вызова

```PHP
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

## Использование корутины

> **Примечание**
> Для использования корутины требуется workerman>=5.0, workerman/redis>=2.0.0 и установка composer require revolt/event-loop ^1.0.0.

```PHP
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

Если не установлены функции обратного вызова, клиент вернет результат асинхронного запроса синхронным способом, и процесс не будет блокирован, что позволяет обрабатывать запросы параллельно.

> **Примечание**
> Использование корутины для psubscribe и subscribe не поддерживается.

## Документация

**Пояснение**

**В режиме обратного вызова обычно есть 2 параметра ($result, $redis), где `$result` - результат, а `$redis` - экземпляр redis. Например:**
```PHP
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// Установка функции обратного вызова для оценки результата set
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// Функции обратного вызова являются опциональными параметрами, здесь приведен пример без функции обратного вызова
$redis->set('key1', 'value1');
// Функции обратного вызова могут быть вложенными
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## Подключение

```PHP
use Workerman\Redis\Client;
// Упущена функция обратного вызова
$redis = new Client('redis://127.0.0.1:6379');
// С функцией обратного вызова
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // Устанавливает тайм-аут соединения 10 секунд, по умолчанию 5 секунд
], function ($success, $redis) {
    // Функция обратного вызова результата соединения
    if (!$success) echo $redis->error();
});
```

## auth

```PHP
// Аутентификация паролем
$redis->auth('password', function ($result) {

});
// Аутентификация по имени пользователя и паролю
$redis->auth('username', 'password', function ($result) {

});
```

## pSubscribe

Подписка на один или несколько каналов, соответствующих заданному шаблону.

Каждый шаблон использует символ \* в качестве маски, например, it\* соответствует всем каналам, начинающимся с it (например, it.news, it.blog, it.tweets и т. д.). news.\* соответствует всем каналам, начинающимся с news. (например, news.it, news.global.today и т. д.).

Примечание: функция обратного вызова pSubscribe имеет 4 параметра ($pattern, $channel, $message, $redis).

После вызова экземпляра $redis метода pSubscribe или subscribe, дальнейшие вызовы этого экземпляра игнорируются.

```PHP
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->psubscribe(['news*', 'blog*'], function ($pattern, $channel, $message) {
    echo "$pattern, $channel, $message"; // news*, news.add, news content
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news.add', 'news content');
});
```

## subscribe

Используется для подписки на определенный или несколько каналов сообщений.

Примечание: функция обратного вызова subscribe имеет 3 параметра ($channel, $message, $redis).

После вызова экземпляра $redis метода pSubscribe или subscribe, дальнейшие вызовы этого экземпляра игнорируются.

```PHP
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->subscribe(['news', 'blog'], function ($channel, $message) {
    echo "$channel, $message"; // news, news content
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news', 'news content');
});
```

## publish

Используется для отправки сообщения в указанный канал.

Возвращает количество получателей сообщения.

```PHP
$redis2->publish('news', 'news content');
```

## select

```PHP
// Упущена функция обратного вызова
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // Параметр select должен быть числом, поэтому $result здесь равен false
    var_dump($result, $redis->error());
});
```

## get

Команда используется для получения значения указанного ключа. Если ключ не существует, возвращает NULL. Если значение ключа не является строкой, возвращает false.

```PHP
$redis->get('key', function($result) {
     // Если ключ не существует, возвращает NULL, а при ошибке - false
    var_dump($result);
});
```

## set

Используется для установки значения для указанного ключа. Если ключ уже содержит другое значение, SET заменит старое, игнорируя тип.

```PHP
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// Третий параметр может содержать время истечения, истекает через 10 секунд
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## setEx, pSetEx

Используется для установки значения и его времени истечения для указанного ключа. Если ключ уже существует, команда SETEX заменит старое значение.

```PHP
// Обратите внимание, что второй параметр содержит время истечения в секундах
$redis->setEx('key', 3600, 'value'); 
// Параметр время в миллисекундах
$redis->pSetEx('key', 3600, 'value'); 
```

## del

Используется для удаления существующих ключей. Возвращает количество ключей, которые были удалены (количеству ключей, не существующих, не присваивается значение).

```PHP
// Удаление одного ключа
$redis->del('key');
// Удаление нескольких ключей
$redis->del(['key', 'key1', 'key2']);
```

## setNx

(SETifNotExists) — используется для установки значения ключа, если ключа не существует.

```PHP
$redis->del('key');
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 1
});
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 0
});
```

## exists

Используется для проверки существования указанного ключа. Возвращает количество существующих ключей.

```PHP
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

## incr, incrBy

Используется для увеличения значения ключа на единицу/указанное значение. Если ключ не существует, ему сначала присваивается значение 0, затем выполняется операция увеличения.

Если значение имеет неверный тип или строковое значение не может быть представлено числом, возвращает false.

Результат успешного увеличения - новое значение.

```PHP
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## incrByFloat

Используется для увеличения значения ключа на указанное дробное значение. Если ключ не существует, ему сначала присваивается значение 0, затем выполняется операция увеличения.

Если значение имеет неверный тип или строковое значение не может быть представлено числом, возвращает false.

Результат успешного увеличения - новое значение.

```PHP
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
});
```
## **decr, decrBy**

Команда уменьшает значение, хранящееся в ключе, на один/указанное значение.
Если ключ не существует, то его значение сначала инициализируется как 0, а затем выполняется операция decr/decrBy.
Если значение имеет неправильный тип или строковое значение не может быть представлено как число, возвращается false.
В случае успеха возвращается уменьшенное числовое значение.
```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

Возвращает значения всех (одного или нескольких) заданных ключей. Если в заданных ключах есть ключ, которого не существует, то он возвращает NULL.
```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```

## **getSet**

Используется для установки значения указанного ключа и возврата старого значения ключа.
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

Случайным образом возвращает ключ из текущей базы данных.
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    }) ;
})
```

## **move**

Перемещает ключ из текущей базы данных в заданную базу данных db.
```php
$redis->select(0);	// переключиться на DB 0
$redis->set('x', '42');	// записать 42 в x
$redis->move('x', 1, function ($result) { 	// переместить в DB 1
    var_dump($result); // 1
}) ;  
$redis->select(1);	// переключиться на DB 1
$redis->get('x', function ($result) {
    var_dump($result); // '42'
}) ;
```

## **rename**

Изменяет имя ключа. Если ключ не существует, возвращается false.
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
}) ;
```

## **renameNx**

Изменяет имя ключа только если новый ключ не существует.
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
}) ;
```

## **expire**

Устанавливает срок действия ключа в секундах. После истечения срока действия ключа он становится недоступным. В случае успеха возвращается 1, если ключ не существует - 0, при возникновении ошибки - false.
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

Команда используется для поиска всех ключей, соответствующих заданному шаблону pattern.
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
}) ;
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
}) ;
```

## **type**

Возвращает тип значения, хранящегося в ключе. Результатом является строка, принимающая одно из значений: string, set, list, zset, hash, none (при отсутствии ключа).
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
}) ;
```

## **append**

Если ключ существует и является строкой, то APPEND добавляет значение к концу существующего значения ключа и возвращает длину строки. Если ключ не существует, то он устанавливается в значение и возвращается длина строки. Если ключ существует, но не является строкой, возвращается false.
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

Получает подстроку строки, хранящейся в определенном ключе. Диапазон подстроки определяется смещениями start и end (включая start и end). Если ключ не существует, возвращается пустая строка. Если ключ не является строкой, возвращается false.
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

Перезаписывает значение строки, хранящееся в указанном ключе, начиная с смещения offset. Если ключ не существует, он устанавливается в указанную строку. Если ключ не является строкой, возвращается false.
Результатом является длина измененной строки.
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

Получает длину строки, хранящейся в указанном ключе. В случае, если ключ хранит не строковое значение, возвращается false.
```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
}) ; 
```

## **getBit**

Получает значение бита в указанном смещении для строки, хранящейся в ключе.
```php
$redis->set('key', "\x7f"); // это 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **setBit**

Устанавливает или очищает значение бита в указанном смещении для строки, хранящейся в ключе.
Результатом является 0 или 1 - значение до изменения.
```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2a = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **bitOp**

Выполняет побитовую операцию над несколькими ключами (содержащими строковые значения) и сохраняет результат в целевом ключе.
Команда BITOP поддерживает четыре побитовых операции: AND, OR, XOR и NOT.
Результатом является размер строки, сохраненной в целевом ключе, равный размеру самой длинной входной строки.
```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
}) ;
```

## **bitCount**

Вычисляет количество установленных битов в строке (счетчик населения).
По умолчанию будет проверяться наличие установленных битов во всех байтах строки. Однако, можно указать операцию подсчета, передав дополнительные параметры *start* и *end* интервала.
Аналогично команде GETRANGE, начало и конец могут содержать отрицательные значения для индексации байтов строки с конца, где -1 является последним байтом, -2 предпоследним и так далее.
Результатом является количество установленных значений битов в строке. Если ключ не существует, возвращает 0.
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

Команда sort используется для сортировки элементов списка, множества и отсортированного множества.
Прототип: `sort($key, $options, $callback);`
Где options является необязательными ключами и значениями:
~~~
$options = [
     'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // или массив шаблонов
    'sort' => 'asc', // или 'desc'
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

Возвращает оставшееся время до истечения ключа в секундах/миллисекундах.
Если у ключа нет ttl, то возвращается -1. Если ключ не существует, то возвращается -2.
```php
$redis->set('key', 'value', 10);
// в секундах
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// в миллисекундах
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// ключ не существует
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```
## **persist**

Удаляет время жизни ключа, чтобы ключ больше не истекал.

Если удаление успешно, возвращает 1, если ключ не существует или у него нет времени жизни, возвращает 0, при ошибке возвращает false.
```php
$redis->persist('key');
```

## **mSet, mSetNx**

Устанавливает несколько пар ключ-значение в одной атомарной команде. mSetNx возвращает 1 только в том случае, если установлены все ключи.

Успешное выполнение возвращает 1, неудача - 0, при ошибке - false.
```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

Устанавливает значение поля в хэш-таблице.

Если поле является новым в хэш-таблице и его значение успешно установлено, возвращает 1. Если поле уже существует в хэш-таблице и его старое значение было перезаписано новым, возвращает 0.

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

Устанавливает значение поля в хэш-таблице, только если поля еще не существует.

Если хэш-таблица не существует, создает новую и выполняет операцию HSET. Если поле уже существует в хэш-таблице, операция игнорируется.

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

Возвращает значение поля в хэш-таблице.

Если указанное поле или ключ не существует, возвращает null.

```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```

## **hLen**

Возвращает количество полей в хэш-таблице.

Если ключ не существует, возвращает 0.

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

Удаляет одно или несколько полей из хэш-таблицы, игнорируя несуществующие поля.

Возвращает количество успешно удаленных полей, не включая игнорируемые поля. Если ключ не является хэшем, возвращает false.

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

Возвращает массив всех полей в хэш-таблице.

Если ключ не существует, возвращает пустой массив. Если ключ не является хэшем, возвращает false.

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

Возвращает массив всех значений полей в хэш-таблице.

Если ключ не существует, возвращает пустой массив. Если ключ не является хэшем, возвращает false.

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

Возвращает ассоциативный массив со всеми полями и их значениями из хэш-таблицы.

Если ключ не существует, возвращает пустой массив. Если ключ не является хэшем, возвращает false.

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
Возвращает
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

## **hExists**

Проверяет, существует ли указанное поле в хэш-таблице. Если существует, возвращает 1, если поле или ключ не существуют, возвращает 0, при ошибке возвращает false.

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## **hIncrBy**

Увеличивает значение поля в хэш-таблице на указанное целочисленное значение.

Увеличение может быть отрицательным, тогда это будет уменьшение значения поля. Если ключ не существует, создает новую хэш-таблицу и выполняет HINCRBY. Если указанное поле не существует, перед выполнением команды значение поля инициализируется как 0.

Для строкового значения поля выполнение  HINCRBY возвращает false.

Значение увеличения ограничено 64-битным знаковым числом.
```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

Аналогично hIncrBy, но значение увеличения - с плавающей точкой.

## **hMSet**

Устанавливает несколько пар поле-значение в хэш-таблице.

Если хэш-таблица не существует, создает пустую и выполняет HMSET.
```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

Возвращает ассоциативный массив значений полей из хэш-таблицы.

Если указанных полей нет в хэш-таблице, соответствующие значения будут null. Если ключ не является хэшем, возвращает false.

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
Вывод:
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```

## **blPop, brPop**

Извлекает и возвращает первый/последний элемент из списка. Если список пуст, блокирует список до тех пор, пока не появится элемент или не будет сделано время на прослушивание.

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

Извлекает последний элемент из одного списка и вставляет его в начало другого списка; если список пуст, блокирует список до тех пор, пока не появится элемент или не будет сделано время на прослушивание. Если время прослушивания истекло, возвращает null.

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

Получает элемент списка по индексу. Можно использовать отрицательные индексы, -1 возвращает последний элемент списка, -2 предпоследний и т. д. Если указанный индекс находится за пределами списка, возвращает null. Если ключ не является списком, возвращает false.

```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

Вставляет элемент в список до или после указанного элемента. Если указанный элемент отсутствует в списке, операция ничего не делает.

Если список не существует, он рассматривается как пустой список, и операция ничего не делает. Если ключ не является списком, возвращает false.

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

Удаляет и возвращает первый элемент списка.

Если ключ не существует, возвращает null.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```
## **lPush (lPush)**

Одно или несколько значений вставляются в начало списка. Если ключ не существует, будет создан пустой список и выполнена операция LPUSH. Если ключ существует, но не является типом списка, возвращается false.

**Примечание:** В Redis версии до 2.4 команда LPUSH принимала только одно значение.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
``` 

## **lPushx (lPushx)**

Добавляет значение в существующий список в начало, операция недействительна, если список не существует, возвращает 0. Если ключ не является типом списка, возвращается false.

Возвращаемое значение после выполнения команды lPushx - длина списка.

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

## **lRange (lRange)**

Возвращает элементы в указанном диапазоне списка, диапазон определяется смещениями START и END. 0 - первый элемент списка, 1 - второй элемент списка и так далее. Можно использовать отрицательные индексы, -1 - последний элемент списка, -2 - предпоследний элемент списка и так далее.

Возвращается массив, содержащий элементы в указанном диапазоне. Возвращается false, если ключ не является типом списка.

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem (lRem)**

Удаляет элементы из списка, равные значению параметра COUNT.

Значения COUNT могут быть следующими:

*   count > 0 : удаляет элементы, равные VALUE, начиная с начала списка и до конца, количество - COUNT.
*   count < 0 : удаляет элементы, равные VALUE, начиная с конца списка и до начала, количество - абсолютное значение COUNT.
*   count = 0 : удаляет все элементы, равные VALUE.

Возвращает количество удаленных элементов. Возвращается 0, если список не существует. Возвращается false, если это не список.

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet (lSet)**

Устанавливает значение элемента по индексу.

В случае успешного выполнения возвращается true, если индекс находится вне диапазона или производится операция LSET для пустого списка, возвращается false.

```php
$redis->lSet('key1', 0, 'X');
```

## **lTrim (lTrim)**

Обрезает список, сохраняя только указанный диапазон элементов, все элементы вне этого диапазона будут удалены.

Индекс 0 означает первый элемент списка, 1 - второй элемент списка и так далее. Можно использовать отрицательные индексы, -1 - последний элемент списка, -2 - предпоследний элемент списка и так далее.

В случае успешного выполнения возвращается true, в случае ошибки - false.

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

## **rPop (rPop)**

Удаляет последний элемент из списка и возвращает его значение.

Если список не существует, возвращается null.

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush (rPopLPush)**

Удаляет последний элемент из списка и добавляет его в начало другого списка, затем возвращает это значение.

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

## **rPush (rPush)**

Вставляет одно или несколько значений в конец списка и возвращает длину списка после вставки.

Если список не существует, будет создан пустой список и выполнена операция RPUSH. Если список существует, но не является типом списка, возвращается false.

**Примечание:** В Redis 2.4 версии, команда RPUSH принимала только одно значение.

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX (rPushX)**

Вставляет значение в конец существующего списка и возвращает длину списка. Если список не существует, операция недействительна, возвращается 0. Если список существует, но не является типом списка, возвращается false.

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen (lLen)**

Возвращает длину списка. Если ключ списка не существует, он интерпретируется как пустой список и возвращает 0. Если ключ не является типом списка, возвращается false.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lLen('key1', function ($r) {
    var_dump($r); // 3
});
```

## **sAdd (sAdd)**

Добавляет один или несколько элементов-членов в множество, существующие члены множества будут проигнорированы.

Если множество ключа не существует, создается множество, содержащее только добавленные элементы в качестве членов.

Если ключ множества не является типом множества, возвращается false.

**Примечание:** В версии Redis 2.4, команда SADD принимала только одно значение-член.

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

## **sCard (sCard)**

Возвращает количество элементов во множестве. Если ключ множества не существует, возвращается 0.

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

## **sDiff (sDiff)**

Возвращает разность между первым множеством и остальными, то есть элементы, отсутствующие в остальных множествах.

Отсутствующие ключи множества считаются пустыми множествами.

```php
$redis->del('s0', 's1', 's2');

...

$redis->sDiff(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // ['2', '4']
});
```

## **sDiffStore (sDiffStore)**

Разность между данными множествами сохраняется в указанном множестве. Если ключ указанного множества уже существует, он будет перезаписан.

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

## **sInter (sInter)**

Возвращает пересечение всех указанных множеств. Если какое-либо из множеств пусто, результатом будет также пустое множество.

```php
$redis->del('s0', 's1', 's2');

$redis...

$redis->sInter(['key1', 'key2', 'key3'], function ($r) {
    var_dump($r); // ['val4', 'val3']
});
```
## **sInterStore**

Сохраняет пересечение между заданными наборами в указанный набор и возвращает количество элементов в сохранённом пересечении. Если указанный набор уже существует, он будет перезаписан.

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

Проверяет, является ли элемент членом набора.

Если элемент является членом набора, то возвращает 1. Если элемент не является членом набора, или ключ не существует, возвращает 0. Если ключ не является набором, возвращает false.

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## **sMembers**

Возвращает все элементы набора. Если набор не существует, возвращает пустой набор.

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## **sMove**

Перемещает элемент из одного набора в другой.

Команда SMOVE является атомарной.

Если исходный набор не существует или не содержит указанный элемент, тогда SMOVE не выполнит никаких действий и вернет 0. В противном случае, элемент будет удален из исходного набора и добавлен в целевой набор.

Если элемент уже существует в целевом наборе, команда SMOVE просто удалит элемент из исходного набора.

Если исходный или целевой набор не является набором, возвращает false.

```php
$redis->sMove('key1', 'key2', 'member13');
```

## **sPop**

Удаляет один или несколько случайных элементов из набора и возвращает их.

Если набор не существует или пуст, возвращает null.

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

Команда Redis Srandmember используется для возврата случайного элемента из набора.

Начиная с версии Redis 2.6, команда Srandmember принимает дополнительный параметр count:

- Если count положительное число и меньше размера набора, команда возвращает массив из count уникальных элементов. Если count больше или равен размеру набора, возвращается весь набор.
- Если count отрицательное число, команда возвращает массив, в котором элементы могут повторяться, а размер массива будет равен абсолютному значению count.

Эта команда подобна SPOP, но SPOP удаляет случайный элемент из набора, тогда как Srandmember просто возвращает случайный элемент без изменения набора.

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

Удаляет один или несколько членов набора, игнорируя несуществующие члены.

Возвращает количество успешно удаленных элементов, за исключением несуществующих элементов. Если ключ не является набором, возвращает false.

До версии 2.4 Redis, команда SREM принимала только одно значение члена.

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

Команда возвращает объединение заданных наборов. Если набор не существует, он считается пустым.

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

Сохраняет объединение заданных наборов в указанный набор и возвращает количество элементов. Если целевой набор уже существует, он будет перезаписан.

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
