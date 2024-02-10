# เว็บแมน-เรดิส

## บทนำ

workeman/redis เป็นคอมโพเนนต์เรดิสแบบไม่เชื่อมต่ออนุญาตใช้งานกับ workerman

> **หมายเหตุ**
> จุดประสงค์หลักของโปรเจกต์นี้คือการทำให้สามารถทำรายการเรดิสแบบไม่เชื่อมต่อ (subscribe, pSubscribe)  เนื่องจากระบบเรดิสเร็วมาก ดังนั้นถ้าไม่มีความจำเป็นในการทำรายการเรดิสแบบไม่เชื่อมต่อ ไม่จำเป็นต้องใช้ไคลเอ็นต์ไม่เชื่อมต่อนี้ การใช้เรดิสเอ็กธเทนชันจะให้ประสิทธิภาพที่ดีกว่า

## การติดตั้ง

```composer require workerman/redis```

## การใช้งานแบบ Callback

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

## การใช้งานแบบ Coroutine

> **หมายเหตุ**
> การใช้งานแบบ Coroutine ต้องใช้ workerman>=5.0, workerman/redis>=2.0.0 และติดตั้ง composer require revolt/event-loop ^1.0.0

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
## **select**
```php
// ข้ามการเรียกคืน
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // ต้องใช้หมายเลขเป็นพารามิเตอร์ select จึงทำให้ $result เป็น false ที่นี่
    var_dump($result, $redis->error());
});
```

## **get**

คำสั่งเพื่อรับค่าของคีย์ที่กำหนด หากคีย์ไม่มีการเก็บไว้ ระบบจะคืนค่า NULL หากค่าที่เก็บไม่ใช่ string ระบบจะคืนค่าเป็น false
```php
$redis->get('key', function($result) {
     // หากคีย์ไม่มีการเก็บไว้ ระบบจะคืนค่า NULL หากเกิดข้อผิดพลาด ระบบจะคืนค่าเป็น false
    var_dump($result);
});
```

## **set**

ใช้เพื่อกำหนดค่าของคีย์ที่กำหนด หากคีย์มีการเก็บค่าแล้ว การใช้ SET จะเขียนทับค่าเดิม โดยไม่สนใจประเภท
```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// พารามิเตอร์ที่สามสามารถส่งเวลาหมดอายุ หลังจาก 10 วินาที
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **setEx, pSetEx**

กำหนดค่าและเวลาหมดอายุของคีย์ที่กำหนด หากคีย์มีอยู่แล้ว คำสั่ง SETEX จะแทนที่ค่าเดิม
```php
// ใส่ใจาการส่งพารามิเตอร์ที่สองเป็นเวลาหมดอายุ หน่วยเป็นวินาที
$redis->setEx('key', 3600, 'value'); 
// pSetEx หน่วยคือมิลลิวินาที
$redis->pSetEx('key', 3600, 'value'); 
```

## **del**

ใช้ลบคีย์ที่มีอยู่ ค่าที่คืนคือจำนวนของคีย์ที่ถูกลบ (คีย์ที่ไม่มีอยู่จะไม่ถูกนับ)
```php
// ลบคีย์หนึ่งคีย์
$redis->del('key');
// ลบหลายคีย์
$redis->del(['key', 'key1', 'key2']);
```

## **setNx**

（**SET**if**N**ot e**X**ists） คำสั่งนี้จะกำหนดค่าของคีย์ที่กำหนดเมื่อคีย์นั้นยังไม่มีอยู่
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

คำสั่งที่ใช้ตรวจสอบว่าคีย์ที่กำหนดมีอยู่หรือไม่ ค่าที่ถูกรีเทิร์นคือจำนวนของคีย์ที่มีอยู่
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

## **incr, incrBy**

เพิ่มค่าที่เก็บในคีย์ไปหนึ่งหรือเพิ่มค่าที่กำหนด เมื่อคีย์ไม่มีอยู่ค่าของคีย์จะถูกกำหนดเป็น 0 และดำเนินการ incr/incrBy
หากค่าเก็บไม่ถูกให้รูปแบบ หรือค่าประเภทสตริงไม่สามารถแสดงเป็นเลข ระบบจะคืนค่าเป็น false
การดำเนินการสำเร็จระบบจะคืนค่าเป็นค่าที่เพิ่มไป
```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **incrByFloat**

เพิ่มค่าที่กำหนดจำนวนทศนิยมเข้าไปในคีย์ที่กำหนด
หากคีย์ไม่มีอยู่ ระบบจะกำหนดค่าของคีย์เป็น 0 และดำเนินการการบวกระบบของการบวกได้
หากค่าเก็บไม่ถูกให้รูปแบบ หรือค่าประเภทสตริงไม่สามารถแสดงเป็นเลข ระบบจะคืนค่าเป็น false
การดำเนินการสำเร็จระบบจะคืนค่าเป็นค่าที่เพิ่มไป
```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## **decr, decrBy**

คำสั่งลดค่าที่เก็บในคีย์ไปหนึ่งหรือค่าที่กำหนด
หากคีย์ไม่มีอยู่ค่าของคีย์จะถูกกำหนดเป็น 0 และดำเนินการ decr/decrBy
```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

คืนค่าของ (คีย์หนึ่งหรือมากกว่าหนึ่ง) คีย์ที่กำหนด หากคีย์ที่กำหนดในคีย์ผ่านมีคีย์ที่ไม่มีอยู่ คำสั่งนี้จะคืนค่าเป็น NULL
```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```

## **getSet**

ใช้สำหรับกำหนดค่าของคีย์ที่กำหนด และคืนค่าเดิมของคีย์
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

คืนค่าของคีย์ที่สุ่มมาจากฐานข้อมูลปัจจุบัน
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    }) ;
})
```

## **move**

ย้ายคีย์ของฐานข้อมูลปัจจุบันไปยังฐานข้อมูลที่กำหนด
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
## **เปลี่ยนชื่อ**

เปลี่ยนชื่อ key โดยถ้า key ไม่มีอยู่จะคืนค่าเป็น false.
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
}) ;
```

## **renameNx**

เปลี่ยนชื่อ key โดยเมื่อ key ใหม่ยังไม่มีอยู่.
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
}) ;
```

## **expire**

ตั้งค่าเวลาหมดอายุของ key หน่วยเป็นวินาที คืนค่าความสำเร็จเป็น 1 ถ้า key ไม่มีอยู่คืนค่าเป็น 0 และกรณีเกิดข้อผิดพลาดคืนค่าเป็น false.
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

ใช้สำหรับค้นหา key ทั้งหมดที่ตรงกับรูปแบบที่กำหนด.
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
}) ;
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
}) ;
```

## **type**

คืนค่าประเภทของข้อมูลที่ key เก็บไว้ ผลลัพธ์ที่ได้เป็น string set list zset hash none โดยที่ none แสดงถึง key ไม่มีอยู่.
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
}) ;
```

## **append**

หาก key มีอยู่และเป็น string คำสั่ง APPEND จะต่อ value เข้าไปที่ท้ายของค่า key และคืนค่าความยาวของ string

หาก key ไม่มีอยู่คำสั่ง APPEND จะตั้งค่า key เป็น value และคืนค่าความยาวของ string

หาก key มีอยู่แต่ไม่ใช่ string จะคืนค่าเป็น false.
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

รับสตริงย่อยที่เก็บอยู่ใน key ที่ระบุ ช่วงการตัดสตริงจะถูกกำหนดโดย start และ end (รวม start และ end) หาก key ไม่มีอยู่ จะคืนค่าเป็นสตริงว่าง หาก key ไม่ใช่ string จะคืนค่าเป็น false.
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

ช่วยเปลี่ยนค่าของ key ที่ระบุเป็นสตริงที่กำหนดไว้ ด้วยการเริ่มต้นจากตำแหน่ง offset ถ้า key ไม่มีอยู่จะตั้งค่า key ตามสตริงที่กำหนดไว้ หาก key ไม่ใช่ string จะคืนค่าเป็น false.

ผลลัพธ์จะเป็นความยาวของสตริงที่เปลี่ยนแปลงไป.
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

รับความยาวของค่าที่เก็บอยู่ใน key ที่ระบุ ถ้า key เก็บค่าไม่ใช่สตริง จะคืนค่าเป็น false.
```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
}) ; 
```

## **getBit**

สำหรับข้อมูลสตริงที่ key เก็บไว้ รับค่า bit ที่อยู่ในตำแหน่ offset ที่ระบุ.
```php
$redis->set('key', "\x7f"); // this is 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **setBit**

สำหรับข้อมูลสตริงที่ key เก็บไว้ ตั้งค่าหรือล้างค่า bit ที่อยู่ในตำแหน่ offset ที่ระบุ.
คืนค่าเป็น 0 หรือ 1 โดยเป็นค่าก่อนการเปลี่ยนแปลง.

```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2a = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **bitOp**

ทำการคำนวณ operation bitwise ในกรณีหลายๆ key (ที่เก็บค่าสตริง) และเก็บผลลัพธ์ไว้ใน key เป้าหมาย.
BITOP command รองรับการทำ operation bitwise มี 4 รูปแบบ ได้แก่ **AND**,**OR**,**XOR** และ**NOT** .
ผลลัพธ์ที่ได้จะเป็นขนาดของสตริงที่เก็บใน key เป้าหมาย ซึ่งเท่ากับขนาดของสตริงที่ยาวที่สุดของ input ที่เข้ามา.
```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
}) ;
```

## **bitCount**

หาค่าบิตที่ตั้งไว้ในสตริง (การนับประชากร) .
โดยปกติแล้วจะตรวจสอบ byte ทั้งหมดในสตริง สามารถระบุการนับได้ในช่วงที่ส่งต่อเพิ่มเติม *start* และ *end* .
อย่างเช่น เริ่มต้นและสิ้นสุดสามารถระบุเป็นค่าลบเพื่อเริ่มจากท้ายสตริง โดยที่ -1 คือ byte สุดท้าย -2 เป็น byte ก่อนสุดท้าย และเป็นอย่างไร.
ผลลัพธ์ที่ได้จะเป็นจำนวนบิตค่า 1 ที่อยู่ในสตริง .
สำหรับ key ที่ไม่มีอยู่จะถือว่าเป็นสตริงว่างโปรด.
```php
$redis->set('key', 'hello');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
}) ;
$redis->bitCount( 'key', function ($result) {
    var_dump($result); //21
}) ;
```
## **การเรียงลำดับ**

คำสั่ง sort สามารถทำการเรียงลำดับสมาชิคใน list、set และ sorted set ได้

Prototype: `sort($key, $options, $callback);`

options สามารถเลือกได้จาก key และ value ต่อไปนี้
```php
$options = [
     'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // หรือเป็นอาร์เรย์ของ pattern
    'sort' => 'asc', // หรือ 'desc'
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

คืนค่าเวลาที่เหลือของ key ในหน่วยวินาที/มิลลิวินาที

ถ้า key ไม่มี ttl จะคืนค่า -1
ถ้า key ไม่มีอยู่จริง จะคืนค่า -2

```php
$redis->set('key', 'value', 10);
// หน่วยเป็นวินาที
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// หน่วยเป็นมิลลิวินาที
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// ไม่มี key อยู่
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```

## **persist**

ลบเวลาหมดอายุของ key ทำให้ key ไม่มีวันหมดอายุ

ถ้าลบสำเร็จจะคืนค่า 1 ถ้า key ไม่มีหรือไม่มีวันหมดอายุจะคืนค่า 0 ถ้าเกิดข้อผิดพลาดจะคืนค่า false.

```php
$redis->persist('key');
```

## **mSet, mSetNx**

เซ็ตค่าของหลาย ๆ key ในคำสั่งที่ออกแบบเพื่อให้สมบูรณ์ mSetNx จะคืนค่าเฉพาะเมื่อทุก ๆ คีย์ถูกเซ็ต

สำเร็จจะคืนค่า 1 หากล้มเหลวจะคืนค่า 0 หากเกิดข้อผิดพลาดจะคืนค่า false.

```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

กำหนดค่าสำหรับฟิลด์ใน hash table

ถ้าฟิลด์เป็นฟิลด์ใหม่ใน hash table และการตั้งค่าสำเร็จ จะคืนค่า 1 หากฟิลด์ใน hash table มีอยู่และค่าเดิมถูกเขียนทับด้วยค่าใหม่ จะคืนค่า 0

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

กำหนดค่าสำหรับฟิลด์ใน hash table ที่ยังไม่ได้ตั้งค่า

ถ้า hash table ไม่มีคีย์ มีการสร้าง hash table ใหม่และดำเนินการ HSET บนคำสั่ง HSETNX ถ้าฟิลด์มีอยู่ใน hash table การดำเนินการจะไม่ถูกต้อง

ถ้า key ไม่มี hash table จะมีการสร้าง hash table ใหม่และดำเนินการ HSETNX

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

คืนค่าของฟิลด์ที่ระบุใน hash table

ถ้าฟิลด์ที่ระบุหรือ key ไม่มีอยู่ จะคืนค่า null

```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```

## **hLen**

ใช้สำหรับการเรียกดูจำนวนของฟิลด์ใน hash table

เมื่อ key ไม่มีอยู่ จะคืนค่า 0

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

ใช้สำหรับการลบฟิลด์ที่ระบุใน hash table หรือหลาย ๆ ฟิลด์ ฟิลด์ที่ไม่มีอยู่จะถูกข้ามไป

คืนค่าจำนวนของฟิลด์ที่ถูกลบสำเร็จ ไม่รวมฟิลด์ที่ถูกข้ามไป ถ้า key ไม่ใช่ hash จะคืนค่า false

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

รับเป็นอาร์เรย์ของฟิลด์ทั้งหมดใน hash table

ถ้า key ไม่มีอยู่ จะคืนค่าอาร์เรย์ว่าง ถ้า key ไม่ใช่ hash จะคืนค่า false

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

คืนค่าในรูปแบบอาร์เรย์ของค่าของฟิลด์ทั้งหมดใน hash table

ถ้า key ไม่มีอยู่ จะคืนค่าอาร์เรย์ว่าง ถ้า key ไม่ใช่ hash จะคืนค่า false

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

คืนค่าในรูปแบบของอาร์เรย์ที่เกี่ยวข้องกันของฟิลด์และค่าทั้งหมดใน hash table

ถ้า key ไม่มีอยู่ จะคืนค่าอาร์เรย์ว่าง ถ้า key ไม่ใช่ hash จะคืนค่า false

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
คืนค่า
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

## **hExists**

ตรวจสอบว่าฟิลด์ที่ระบุใน hash table มีหรือไม่ ถ้ามีจะคืนค่า 1 ถ้าฟิลด์หรือ key ไม่มีจะคืนค่า 0 ถ้าเกิดข้อผิดพลาดจะคืนค่า false

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```
## **hIncrBy**

ใช้สำหรับเพิ่มค่าของฟิลด์ในตารางแฮชด้วยจำนวนเพิ่มที่ระบุ

จำนวนเพิ่มก็สามารถเป็นค่าลบได้ที่ทำให้มีการดำเนินการการลบของฟิลด์ที่ระบุ 

หากคีย์ในตารางแฮชไม่มีอยู่ การใช้คำสั่ง HINCRBY จะสร้างตารางแฮชใหม่และดำเนินการ HINCRBY 

หากระบุฟิลด์ที่ไม่มีอยู่ ค่าของฟิลด์จะถูกกำหนดเริ่มต้นเป็น 0 ก่อนการดำเนินการคำสั่ง

การใช้คำสั่ง HINCRBY กับฟิลด์ที่เก็บค่าเป็นสตริงจะคืนค่า false

ค่าของการดำเนินการถูก จำกัดไว้ใน 64 บิตของตัวเลขที่เป็นจำนวนเต็มที่มีเครื่องหมาย
```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```


## **hIncrByFloat**

คล้ายกับ hIncrBy แต่ทำการเพิ่มจำนวนเป็นทศนิยม

## **hMSet**

ทำการตั้งค่าทั้งหมดของค่า สำหรับคู่ field-value (ฟิลด์-ค่า) ในตารางแฮช

คำสั่งนี้จะทับค่าฟิลด์ที่มีอยู่แล้วในตารางแฮช

หากรายการแฮชไม่มี ตัวภายในจะทำการสร้างตารางแฮชว่างและดำเนินการ HMSET

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

คืนค่าในรูปแบบของอาร์เรย์แบบสมาชิกด้วยค่าฟิลด์ในตารางแฮชหนึ่งหรือหลาย ๆ อันที่ระบุ

หากฟิลด์ที่ระบุไม่มีอยู่ในตารางแฮช ค่าที่กำหนดก็จะเป็นค่า null ถ้า key ไม่ใช่แอช จะคืนค่าเป็น false
```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```

ผลลัพธ์
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```
## **blPop, brPop**

ดึงออกและรับค่าสมาชิกแรก/สมาชิกสุดท้ายของรายการ หากรายการไม่มีสมาชิก ระบบจะบล็อกรายการไว้จนกว่าจะหมดเวลาหรือพบว่าสามารถดึงออกสมาชิกได้

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

นำสมาชิกสุดท้ายของรายการไปและแทรกไว้ที่หัวของรายการอื่น ๆ หากรายการไม่มีสมาชิก ระบบจะบล็อกรายการไว้จนกว่าจะหมดเวลาหรือพบว่าสามารถดึงออกสมาชิกได้ หากระบบหมดเวลาจะคืนค่าเป็น null

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

ใช้ดึงสมาชิกในรายการด้วยดัชนี คุณยังสามารถใช้ดัชนีที่เป็นค่าลบ เช่น -1 หมายถึงสมาชิกสุดท้ายของรายการ -2 หมายถึงสมาชิกก่อนสุดท้ายของรายการและให้ต่อเนื่องไปเรื่อย ๆ  

หากดัชนีที่กำหนดไม่อยู่ในช่วงของรายการ จะคืนค่า null ถ้า key ที่กำหนดไม่ใช่ชนิดรายการ จะคืนค่าเป็น false

```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

แทรกสมาชิกในรายการก่อนหรือหลังสมาชิกที่กำหนด ถ้าสมาชิกที่กำหนดไม่มีอยู่ในรายการ จะไม่ทำการดำเนินการใด ๆ 

ถ้ารายการไม่มีอยู่ จะถือว่าเป็นรายการว่างไม่ทำการดำเนินการใด ๆ 

หาก key ไม่ใช่ชนิดของรายการ จะคืนค่าเป็น false
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

ดึงออกและคืนค่าสมาชิกแรกของรายการ

เมื่อ key ไม่มีอยู่ จะคืนค่าเป็น null
```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```


## **lPush**

แทรกหนึ่งหรือหลายค่าเข้าที่หัวของรายการ ถ้า key ไม่มีอยู่ จะสร้างรายการว่างไว้แล้วดำเนินการ LPUSH

เมื่อ key มีอยู่ แต่ไม่ใช่ชนิดของรายการ จะคืนค่าเป็น false

**หมายเหตุ:** คำสั่ง LPUSH ในรีดิสเวอร์ชัน 2.4 ก่อนหน้านี้ รับเฉพาะค่าเดี่ยวเท่านั้น
```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```


## **lPushx**

แทรกค่าเข้าไปที่หัวของรายการที่มีอยู่แล้ว ถ้ารายการไม่มีอยู่จะไม่ทำการดำเนินการและคืนค่าเป็น 0 ถ้า key ไม่ใช่ชนิดของรายการจะคืนค่าเป็น false

ค่าที่คืนจากการดำเนินการ lPushx คือความยาวของรายการหลังจากการดำเนินการ lPushx
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

คืนค่าอาร์เรย์ขององค์ประกอบในช่วงที่ระบุในลิสต์ โดยใช้ offset START และ END ที่ระบุ เมื่อ 0 หมายถึงองค์ประกอบแรกของลิสต์ 1 หมายถึงองค์ประกอบที่สองของลิสต์ และต่อไปนี้ คุณยังสามารถใช้ดัชนีลบได้ โดย -1 หมายถึงองค์ประกอบสุดท้ายของลิสต์ -2 หมายถึงองค์ประกอบที่นับถอยหลังที่สองของลิสต์ และอื่น ๆ

คืนค่าอาร์เรย์ขององค์ประกอบในช่วงที่ระบุ หาก key ไม่ใช่ชนิดลิสต์จะคืนค่าเป็นเท็จ

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

ลบองค์ประกอบในลิสต์ที่มีค่าเท่ากับอาร์กิวเมนต์ COUNT

ค่า COUNT สามารถเป็นได้เป็นค่าต่อไปนี้

*   count > 0 : ค้นหาจากหัวของลิสต์ไปทางหาง ลบองค์ประกอบที่มีค่าเท่าเท่ากับ VALUE จำนวน COUNT 
*   count < 0 : ค้นหาจากทางหาง ลิสต์ไปหัว ลบองค์ประกอบที่มีค่าเท่ากับ VALUE จำนวนเป็นค่าสัมบูรณ์ของ COUNT 
*   count = 0 : ลบทุกองค์ประกอบที่มีค่าเท่ากับ VALUE ออกจากลิสต์

คืนค่าจำนวนองค์ประกอบที่ถูกลบ คืนค่าเป็น 0 เมื่อลิสต์ไม่มีอยู่ และเมื่อไม่ใช่ชนิดลิสต์จะคืนค่าเป็นเท็จ

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

ตั้งค่าขององค์ประกอบด้วยดัชนี

คืนค่าเท็จเมื่อดัชนีเกินขอบเขต หรือ lset โอกาสว่างเปล่า

```php
$redis->lSet('key1', 0, 'X');
```

## **lTrim**

ตัดตัดลิสต์ทำให้ลิสต์เก็บเฉพาะองค์ประกอบที่อยู่ในช่วงที่ระบุ องค์ประกอบที่ไม่อยู่ในช่วงจะถูกลบออก

ดัชนี 0 หมายถึงองค์ประกอบแรกของลิสต์ และ 1 หมายถึงองค์ประกอบที่สองของลิสต์ และต่อมา คุณยังสามารถใช้ดัชนีลบ หาก -1 หมายถึงองค์ประกอบสุดท้ายของลิสต์ -2 หมายถึงองค์ประกอบที่นับถอยหลังจากที่สองของลิสต์ และอื่น ๆ

คืนค่าเป็นเท็จเมื่อไม่สำเร็จ

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

ใช้เพื่อลบองค์ประกอบสุดท้ายของลิสต์และคืนค่าขององค์ประกบที่ถูกลบ

เมื่อลิสต์ไม่มีอยู่ จะคืนค่าเป็น null

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush**

ใช้เพื่อลบองค์ประกบสุดท้ายของลิสต์และเพิ่มองค์ประกอบนั้นไปยังลิสต์อีกลิสต์หนึ่งและคืนค่า

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

เพิ่มหรือหลายานองค์ประกบเข้าไปที่ท้ายลิสต์ (ทางขวา) และคืนค่าของลิสต์หลังจากการเพิ่ม เมื่อลิสต์ไม่มีอยู่ ระบบจะสร้างลิสต์เปล่าและทำการ RPUSH หากลิสต์ไม่มีอยู่แต่ไม่ใช่ชนิดลิสต์จะคืนค่าเป็นเท็จ

**หมายเหตุ：** ในเวอร์ชั่น Redis 2.4 ทุกคำสั่ง RPUSH จะรับค่าเพียงหนึ่งค่าเท่านั้น

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX**

เพิ่มค่าเข้าไปที่ท้ายลิสต์ที่มีอยู่และคืนค่าของลิสต์หลังจากการเพิ่ม หากลิสต์ไม่มีอยู่ ระบบจะไม่ทำการเพิ่มและคืนค่าเป็น 0 เมื่อลิสต์ไม่มีอยู่ แต่คืนค่าเป็นเท็จเมื่อไม่ใช่ชนิดลิสต์

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen**

คืนค่าความยาวของลิสต์ หาก key ไม่มีอยู่ระบบจะตีค่าแทนที่ด้วยลิสต์เปล่า เมื่อ key ไม่ใช่ชนิดลิสต์ ระบบจะคืนค่าเป็นเท็จ

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

เพื่อเพิ่มหนึ่งหรือหลายานัดของสมาชิกเข้าไปยังเซ็ต สมาชิกที่มีอยู่แล้วจะถูกข้ามเวลาการเพิ่ม

หาก key ของเซ็ตยังไม่มีอยู่ ระบบจะสร้างเซ็ตที่มีสมาชิกเพียงอย่างเดียวที่เพิ่มเข้าไป

เมื่อ key ของเซ็ตไม่ใช่ชนิดเซ็ต ระบบจะคืนค่าเป็นเท็จ

**หมายเหตุ：** ในเวอร์ชั่น Redis2.4 คำสั่ง SADD จะรับค่าเพียงหนึ่งค่าเท่านั้น

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

คืนค่าจำนวนสมาชิกในเซ็ต หากคีย์ของเซ็ตไม่มี คืนค่า 0

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

คืนค่าเฉพาะสมาชิกที่มีในเซ็ตแรก แต่ไม่มีในเซ็ตอื่น ๆ ถ้าเซ็ตที่กำหนดไม่มี จะถือว่าเป็นเซ็ตว่าง

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

เก็บค่าตัวต่างของเซ็ตที่กำหนดไว้ในเซ็ตที่กำหนด หากเซ็ตที่กำหนดมีอยู่แล้ว จะถูกเคลียร์และเขียนทับ

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

คืนค่าเซ็ตที่เป็นส่วนร่วมของเซ็ตทุกตัวที่กำหนด ถ้าเซ็ตที่กำหนดมีเป็นเซ็ตว่าง ผลลัพธ์ก็จะเป็นเซ็ตว่าง

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

เก็บเซ็ตส่วนร่วมของเซ็ตที่กำหนดไว้ในเซ็ตที่กำหนดและคืนค่าจำนวนสมาชิกของเซ็ตที่เก็บส่วนร่วม หากเซ็ตที่กำหนดมีอยู่แล้ว ก็จะถูกเคลียร์และเขียนทับ

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

ตรวจสอบว่าสมาชิกที่ระบุเป็นสมาชิกของเซ็ตหรือไม่

หากสมาชิกเป็นสมาชิกของเซ็ต คืนค่า 1 
หากสมาชิกไม่ได้เป็นสมาชิกของเซ็ต หรือ คีย์ไม่มี คืนค่า 0 
หาก key ไม่ใช่ชนิดของเซ็ต คืนค่า false

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## **sMembers**

คืนค่าสมาชิกทั้งหมดในเซ็ต ถ้าเซ็ตไม่มี ก็ถือว่าเป็นเซ็ตว่าง

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## **sMove**

ย้ายสมาชิกที่ระบุจากเซ็ตต้นทางไปยังเซ็ตปลายทาง

SMOVE เป็นการกระทำแบบอะตอม

หากเซ็ตต้นทางไม่มีหรือไม่มีสมาชิกที่ระบุ SMOVE จะไม่ทำการใด ๆ และคืนค่าเพียง 0
มิฉะนั้น สมาชิกที่ระบุจากเซ็ตต้นทางจะถูกลบออก และเพิ่มไปยังเซ็ตปลายทาง

เมื่อเซ็ตต้นทางหรือเซ็ตปลายทางไม่ใช่ชนิดของเซ็ต จะคืนค่า false

```php
$redis->sMove('key1', 'key2', 'member13');
```

## **sPop**

นำสมาชิกจำนวนหนึ่งหรือหลายสมาชิกออกจากเซ็ต และคืนค่าสมาชิกที่ถูกนำออก

หากเซ็ตไม่มีหรือเป็นเซ็ตว่าง คืนค่า null

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

คำสั่ง Redis Srandmember ใช้สำหรับส่งค่าสมาชิกที่สุ่มมาจากเซ็ต

ตั้งแต่เวอร์ชัน Redis 2.6 เป็นต้นมา Srandmember สามารถรับพารามิเตอร์ count ได้:
- ถ้า count เป็นจำนวนเต็มบวก และน้อยกว่าขนาดของเซ็ต คำสั่งจะส่งค่ากลับเป็นอาร์เรย์ที่มีจำนวนตัวแสดงออกมา count โดยที่ค่าภายในอาร์เรย์ไม่ซ้ำกัน ถ้า count มากกว่าหรือเท่ากับขนาดของเซ็ต จะส่งค่ากลับเป็นเซ็ตทั้งหมด
- ถ้า count เป็นจำนวนเต็มลบ คำสั่งจะส่งค่ากลับเป็นอาร์เรย์ ค่าภายในอาร์เรย์อาจปรากฏซ้ำซ้อนหลายครั้ง และความยาวของอาร์เรย์คือค่าสัมบูรณ์ของ count

การกระทำนี้คล้ายกับ SPOP แต่ SPOP จะลบสมาชิกที่สุ่มมาจากเซ็ตและส่งค่ากลับ ในขณะที่ Srandmember เพียงแค่ส่งค่าสมาชิกที่สุ่มมาจากเซ็ตโดยไม่มีการเปลี่ยนแปลงใด ๆ

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

ลบสมาชิกหนึ่งหรือมากกว่าหนึ่งสมาชิกจากเซ็ต สมาชิกที่ไม่มีอยู่จะถูกข้าม

ส่งค่ากลับเป็นจำนวนของสมาชิกที่ถูกลบสำเร็จโดยไม่รวมที่ถูกข้าม

เมื่อ key ไม่ใช่ชนิดของเซ็ต จะส่งค่ากลับเป็น false

ก่อนเวอร์ชัน Redis 2.4 เป็นต้นมา SREM รับเฉพาะค่าสมาชิกเดี่ยวเท่านั้น

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

คำสั่งส่งค่ากลับเป็นสมาชิกรวมของเซ็ตที่กำหนด ถ้าไม่มี key เซ็ตที่กำหนดจะถือว่าเป็นเซ็ตว่าง

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

บันทึกสมาชิกรวมของเซ็ตที่กำหนดไว้ในเซ็ตหมายเลขหนึ่งที่กำหนดและส่งค่ากลับเป็นจำนวนของสมาชิก หากเซ็ตหมายเลขหนึ่งที่กำหนดมีอยู่แล้ว จะทำการเขียนทับ

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
