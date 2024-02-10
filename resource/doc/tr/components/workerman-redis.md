# webman-redis

## Giriş

webeman/redis, workerman'ın asenkron redis bileşenine dayalıdır.

> **Not**
> Bu proje, özellikle redis asenkron aboneliği (subscribe, pSubscribe) gerçekleştirmeyi amaçlamaktadır.
> Redis oldukça hızlı olduğu için, pSubscribe subscribe asenkron aboneliği gerekmeyen durumlarda, bu asenkron istemciyi kullanmanıza gerek yoktur, redis uzantısını kullanmak daha iyi performans sağlar.

## Kurulum:

```Composer require workerman/redis```

## Geriçağırım Kullanımı

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

## Coroutine Kullanımı

> **Not**
> Coroutine kullanımı workerman>=5.0 ve workerman/redis>=2.0.0 gerektirir ve composer require revolt/event-loop ^1.0.0 kurulu olmalıdır.

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

Gerçekleştirilmeyen geriçağırım fonksiyonları olmadığında, istemci, asenkron istek sonuçlarını senkron bir şekilde döndürür ve istek süreci mevcut süreci bloke etmez, yani eşzamanlı olarak istekleri işleyebilir.

> **Not**
> psubscribe subscribe, coroutine kullanımını desteklemez.

# Dokümantasyon
**Notlar**

**Geriçağırım yönteminde, geriçağırım fonksiyonları genellikle iki parametreye sahiptir ($result, $redis), `$result` sonuç ve `$redis` redis örneğidir. Örneğin:**

```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// Geri çağırım fonksiyonunu ayarlama ile set çağrısını değerlendirme
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// Geri çağırım fonksiyonları her zaman isteğe bağlı parametrelerdir, burada geri çağrım fonksiyonu atlanmıştır
$redis->set('key1', 'value1');
// Geri çağırım fonksiyonları iç içe olabilir
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **Bağlantı**
```php
use Workerman\Redis\Client;
// Geri çağırım atlandı
$redis = new Client('redis://127.0.0.1:6379');
// Geri çağırımla birlikte
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // Bağlantı zaman aşımını 10 saniye olarak ayarla, varsayılan 5 saniye
], function ($success, $redis) {
    // Bağlantı sonucu geri çağırımı
    if (!$success) echo $redis->error();
});
```

## **auth**
```php
// Şifre doğrulama
$redis->auth('password', function ($result) {
    
});
// Kullanıcı adı ve şifre doğrulama
$redis->auth('username', 'password', function ($result) {

});
```

## **pSubscribe**

Belirli bir modelle eşleşen bir veya daha fazla kanalı abone etmek için kullanılır.

Her model, \* ile eşleşen karakter olarak kullanılır, örneğin it\* it ile başlayan tüm kanallara eşleşir (it.news, it.blog, it.tweets vb.). news.\* news. ile başlayan tüm kanallara eşleşir (news.it, news.global.today vb.). vb.

Not: pSubscribe geri çağırım fonksiyonu 4 parametreye sahiptir ($pattern, $channel, $message, $redis)

$redis örneği pSubscribe veya subscribe ara yüzünü çağırdıktan sonra, bu örnek başka bir yöntem çağırıldığında göz ardı edilir.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->psubscribe(['news*', 'blog*'], function ($pattern, $channel, $message) {
    echo "$pattern, $channel, $message"; // news*, news.add, news içerik
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news.add', 'news içerik');
});
```

## **subscribe**

Belirli olarak verilen bir veya daha fazla kanalı abone etmek için kullanılır.

Not: subscribe geri çağırım fonksiyonu 3 parametreye sahiptir ($channel, $message, $redis)

$redis örneği pSubscribe veya subscribe ara yüzünü çağırdıktan sonra, bu örnek başka bir yöntem çağırıldığında göz ardı edilir.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->subscribe(['news', 'blog'], function ($channel, $message) {
    echo "$channel, $message"; // news, news içerik
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news', 'news içerik');
});
```

## **publish**

Belirli bir kanala bilgi göndermek için kullanılır.

Bilgiyi alan abonelerin sayısını döndürür.

```php
$redis2->publish('news', 'news içerik');
```

## **select**
```php
// Geri çağırım atlandı
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // seçim parametresi bir numara olmalıdır, bu nedenle burada $result false olacaktır
    var_dump($result, $redis->error());
});
```

## **get**

Belirtilen anahtarın değerini almak için kullanılır. Eğer anahtar mevcut değilse, NULL döner. Eğer anahtarın sakladığı değer bir dize türü değilse, false döner.
```php
$redis->get('key', function($result) {
    // Anahtar yoksa NULL döner, hata olursa false döner
    var_dump($result);
});
```

## **set**

Verilen anahtarın değerini ayarlamak için kullanılır. Eğer anahtar zaten varsa, SET eski değeri geçersiz kılar ve türüne bakılmaksızın ezer.
```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// Üçüncü parametrele geçerli zamanı iletebilirsiniz, 10 saniye sonra geçerlilik süresi dolar
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **setEx, pSetEx**

Belirtilen anahtarın değerini ve geçerlilik süresini ayarlamak için kullanılır. Eğer anahtar zaten varsa, SETEX komutu eski değeri değiştirir.
```php
// İkinci parametre geçerlilik süresi iletilmelidir, saniye cinsinden
$redis->setEx('key', 3600, 'value'); 
// pSetEx birim olarak milisaniye alır
$redis->pSetEx('key', 3600, 'value'); 
```

## **del**

Mevcut anahtarı silmek için kullanılır, sonuç olarak silinen anahtar sayısını döner (var olmayan anahtarlar sayılmaz).
```php
// Bir anahtarı sil
$redis->del('key');
// Birden çok anahtarı sil
$redis->del(['key', 'key1', 'key2']);
```

## **setNx**

Belirtilen anahtar henüz mevcut değilse, anahtarın belirtilen değerini ayarlamak için kullanılır.
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

Belirtilen anahtarın mevcut olup olmadığını kontrol etmek için kullanılır. Sonuç olarak mevcut anahtar sayısını döner.
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

Belirtilen anahtarın değerini bir artırır/belirtilen değeri ekler. Eğer anahtar mevcut değilse, anahtarın değeri öncelikle 0 olarak ayarlanır ve daha sonra incr/incrBy işlemi gerçekleştirilir.
Eğer değeri yanlış bir türde ise veya dize türündeki değer bir sayıyı temsil edemiyorsa, false döner.
Başarılı ise artırılan değeri döner.
```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **incrByFloat**

Belirtilen anahtarın değerine belirtilen kayan noktalı sayıyı ekler.
Eğer anahtar mevcut değilse, anahtarın değeri öncelikle 0 olarak ayarlanır ve daha sonra eklem işlemi gerçekleştirilir.
Eğer değeri yanlış bir türde ise veya dize türündeki değer bir sayıyı temsil edemiyorsa, false döner.
Başarılı ise artırılan değeri döner.
```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```
## **decr, decrBy**

Komut, key'in depoladığı değeri bir/verilen azaltma miktarıyla azaltır.
Eğer key mevcut değilse, key'in değeri öncelikle 0 olarak başlatılır ve sonra decr/decrBy işlemi gerçekleştirilir.
Değer yanlış bir tipe sahipse veya string tipindeki değer bir sayıya dönüştürülemezse false döner.
Başarıyla azaltılan sayı değerini döndürür.
```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

Verilen key'lerin tümünün(tek veya birden fazla) değerlerini döndürür. Eğer verilen key'lerden biri mevcut değilse, o key NULL olarak döner.
```php
$redis->set('key1', 'değer1');
$redis->set('key2', 'değer2');
$redis->set('key3', 'değer3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'değer1', null];
}); 
```

## **getSet**

Belirli bir key'in değerini ayarlamak ve key'in eski değerini döndürmek için kullanılır.
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

Mevcut veritabanından rastgele bir key döndürür.
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    }) ;
})
```

## **move**

Mevcut veritabanındaki bir key'i belirtilen db veritabanına taşır.
```php
$redis->select(0);	// DB 0'a geçiş
$redis->set('x', '42');	// x'e 42 yaz
$redis->move('x', 1, function ($result) { 	// DB 1'e taşı
    var_dump($result); // 1
}) ;  
$redis->select(1);	// DB 1'e geçiş
$redis->get('x', function ($result) {
    var_dump($result); // '42'
}) ;
```

## **rename**

Key'in adını değiştirir, key mevcut değilse false döner.
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
}) ;
```

## **renameNx**

Yeni key mevcut değilse key'in adını değiştirir.
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
}) ;
```

## **expire**

Key'in süresini ayarlar, süre bittiğinde key kullanılamaz. Birim saniyedir. Başarılıysa 1 döner, key mevcut değilse 0, hata olursa false döner.
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

Belirli model patternine uyan tüm key'leri bulur.
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
}) ;
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
}) ;
```

## **type**

Key'in depoladığı değerin tipini döndürür. Sonuç, string, set, list, zset, hash, none'den biridir; none, key'in mevcut olmamasını ifade eder.
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
}) ;
```

## **append**

Eğer key mevcutsa ve bir string ise, APPEND komutu value'yu key'in mevcut değerinin sonuna ekler ve string uzunluğunu döndürür.
Eğer key mevcut değilse, APPEND basitçe verilen key'i value olarak ayarlar, SET key value gibi davranır ve string uzunluğunu döndürür.
Eğer key mevcut ise ama bir string değilse, false döner.
```php
$redis->set('key', 'değer1');
$redis->append('key', 'değer2', function ($result) {
    var_dump($result); // 12
}) ; 
$redis->get('key', function ($result) {
    var_dump($result); // 'değer1değer2'
}) ;
```

## **getRange**

Belirtilen key'de depolanan string'in belirli bir alt dizisini alır. String'in kesme aralığı start ve end konumlarına göre belirlenir (start ve end dahil). Eğer key mevcut değilse, boş bir string döner. Key string tipinde değilse false döner.
```php
$redis->set('key', 'string değer');
$redis->getRange('key', 0, 5, function ($result) {
    var_dump($result); // 'string'
}) ; 
$redis->getRange('key', -5, -1 , function ($result) {
    var_dump($result); // 'değer'
}) ;
```

## **setRange**

Belirtilen key'de depolanan string değerini belirtilen bir string ile değiştirir, değişiklik offset konumundan başlar. Eğer key mevcut değilse, verilen string olarak key'i ayarlar. Key bir string değilse false döner.

Sonuç olarak değişen string uzunluğunu döndürür.
```php
$redis->set('key', 'Merhaba dünya');
$redis->setRange('key', 6, "redis", function ($result) {
    var_dump($result); // 11
}) ; 
$redis->get('key', function ($result) {
    var_dump($result); // 'Merhaba redis'
}) ; 
```

## **strLen**

Belirtilen key'de depolanan string değerinin uzunluğunu alır. Key'in depoladığı bir string değilse, false döner.
```php
$redis->set('key', 'değer');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
}) ; 
```

## **getBit**

Key'de depolanan string değerinde, belirtilen konumda biti alır.
```php
$redis->set('key', "\x7f"); // bu 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **setBit**

Key'de depolanan string değerinde, belirtilen konumda biti ayarlar veya temizler.
Değişiklik öncesi değeri 0 veya 1 olarak döner.
```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2a = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **bitOp**

Birden fazla key (string değerler içeren) arasında bitsel operasyonu gerçekleştirir ve sonucu hedef keye depolar.

BITOP komutu dört bitsel işlemi destekler: **AND**, **OR**, **XOR** ve **NOT**.

Sonuç, hedef key'de depolanan stringin boyutudur, yani en uzun giriş stringinin boyutuna eşittir.
```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
}) ;
```

## **bitCount**

String içinde ayarlanmış bit sayısını (nüfus sayımı) hesaplar.

Varsayılan olarak, tüm baytları kontrol eder. Yalnızca ek parametreler *start* ve *end* aralığında belirli bir sayım yapılabilir.

GETRANGE komutu gibi, başlangıç ve bitiş negatif değerler içerebilir, böylece dizeyi tersten indeksleyebilir, -1 son baytı, -2 sondan ikinci karakteri temsil eder, vb.

Sonuç, string içinde 1 olan bit sayısını döndürür.

Mevcut olmayan bir anahtar boş bir dize olarak ele alındığından, bu komut sıfırı döndürür.
```php
$redis->set('key', 'merhaba');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
}) ;
$redis->bitCount( 'key', function ($result) {
    var_dump($result); //21
}) ;
```

## **sort**

sort komutu, list, set ve sıralı setin öğelerini sıralamak için kullanılır.

Prototip: `sort($key, $options, $callback);`

options aşağıdaki isteğe bağlı key ve value'ları içerir
~~~
$options = [
     'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // veya desen dizisi
    'sort' => 'asc', // veya 'desc'
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

Key'in kalan süresini saniye/milisaniye cinsinden döndürür.

Eğer key'in ttl'si yoksa -1 döner. Eğer key mevcut değilse -2 döner.
```php
$redis->set('key', 'değer', 10);
// saniye cinsinden
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// milisaniye cinsinden
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// key mevcut değil
$redis->pttl('key-yok', function ($result) {
    var_dump($result); // -2
});
```
## **persist**

Belirtilen anahtarın son kullanma süresini kaldırarak, anahtarın süresiz olarak kalmasını sağlar.

Başarıyla kaldırılırsa 1 döner; anahtar mevcut değilse veya son kullanma süresi yoksa 0 döner; hata oluşursa false döner.
```php
$redis->persist('key');
```

## **mSet, mSetNx**

Birden çok anahtar-değer çiftini atomik bir komutla ayarlar. mSetNx, tüm anahtarlar ayarlandığında sadece 1 döner.

Başarılıysa 1, başarısızsa 0 döner; hata oluşursa false döner.
```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

Hash tablosundaki bir alana değer atar.

Eğer alan yeni bir alansa ve değer başarılı bir şekilde ayarlandıysa 1 döner. Eğer hash tablosundaki alan zaten mevcutsa ve eski değeri yeni değerle değiştirildiyse 0 döner.

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

Hash tablosunda bulunmayan bir alana değer atar.

Eğer hash tablosu mevcut değilse, yeni bir hash tablosu oluşturulur ve HSET işlemi gerçekleştirilir.

Eğer alan hash tablosunda zaten mevcutsa, işlem geçersiz olur.

Eğer anahtar mevcut değilse, yeni bir hash tablosu oluşturulur ve HSETNX komutu gerçekleştirilir.
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

Belirtilen hash tablosundaki alana ait değeri döndürür.

Verilen alan veya anahtar mevcut değilse, null döner.
```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```

## **hLen**

Hash tablosundaki alan sayısını almak için kullanılır.

Anahtar mevcut değilse, 0 döner.
```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

Komut, belirtilen alanları bir hash tablosundan siler. Mevcut olmayan alanlar görmezden gelinir.

Başarıyla silinen alanların sayısını döner; göz ardı edilen alanları döndürmez. Eğer anahtar bir hash dışıysa false döner.
```php
$redis->hDel('h', 'key1');
```

## **hKeys**

Hash tablosundaki tüm alanları bir dizi olarak alır.

Eğer anahtar mevcut değilse, boş bir dizi döner. Eğer anahtar hash türünde değilse, false döner.
```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

Hash tablosundaki tüm alanların değerlerini bir dizi olarak döndürür.

Eğer anahtar mevcut değilse, boş bir dizi döner. Eğer anahtar hash türünde değilse, false döner.
```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

Hash tablosundaki tüm alanları ve değerlerini ilişkili dizi olarak döndürür.

Eğer anahtar mevcut değilse, boş bir dizi döner. Eğer anahtar hash türünde değilse, false döner.
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
Çıktı
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

## **hExists**

Hash tablosundaki belirtilen alanın var olup olmadığını kontrol eder.

Eğer varsa 1, alan veya anahtar mevcut değilse 0, hata oluşursa false döner.
```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## **hIncrBy**

Hash tablosundaki alanın değerine belirtilen artış miktarını ekler.

Artış miktarı negatif olabilir, belirtilen alanın değerini azaltır. 

Eğer anahtar mevcut değilse, yeni bir hash tablosu oluşturulur ve HINCRBY komutu gerçekleştirilir.

Belirtilen alan mevcut değilse, komuttan önce alanın değeri sıfırlanır.

Bir dize değeri saklayan alana HINCRBY komutu uygulandığında false döner.

Bu işlemin değeri 64 bit işaretli sayı temsili içinde sınırlandırılmıştır.
```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

hIncrBy ile benzerdir, ancak artış miktarı ondalıklı sayıdır.

## **hMSet**

Birden çok alan-değer çiftini hash tablosuna ayarlar.

Bu komut, hash tablosundaki var olan alanları üzerine yazar.

Eğer hash tablosu mevcut değilse, boş bir hash tablosu oluşturur ve HMSET işlemi gerçekleştirilir.
```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

Hash tablosundaki belirtilen alanların değerini ilişkili dizi olarak döndürür.

Eğer belirtilen alanlardan herhangi biri hash tablosunda mevcut değilse, karşılık gelen alana null değeri atanır. Eğer anahtar bir hash değilse, false döner.
```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
Çıktı
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```

## **blPop, brPop**

Listenin ilk/son elemanını kaldırır ve alır. Liste boşsa, bekleyerek belirli bir süreye veya çıkarılabilir bir elemanın bulunmasına kadar bekler.

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

Listeden son elemanı alır ve diğer listeye ilk eleman olarak ekler; Liste boşsa, belirli bir süreye veya çıkarılabilir bir elemanın bulunmasına kadar bekler. Süre aşılarsa null döner.

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

Listedeki bir elemanı indeks numarasına göre alır. Negatif indekslerde -1 son elemanı, -2 sondan bir önceki elemanı ifade eder.

Belirtilen indeks liste sınırlarının dışındaysa null döner. Anahtar bir liste türünde değilse, false döner.
```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

Listede belirli bir elemanın öncesine veya sonrasına yeni eleman ekler. Belirtilen eleman liste içinde yoksa herhangi bir işlem yapılmaz.

Belirtilen liste yoksa, boş bir liste olarak kabul edilir, herhangi bir işlem yapılmaz.

Eğer anahtar bir liste türünde değilse, false döner.
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

Listeden ilk elemanı kaldırır ve döndürür.

Eğer liste mevcut değilse, null döner.
```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```
## **lPush**

Bir veya daha fazla değeri listenin başına ekler. Eğer key mevcut değilse, LPUSH işlemi için boş bir liste oluşturulur. Key mevcut ancak liste türünde değilse false döner.

**Not:** Redis 2.4 sürümünden önceki LPUSH komutları yalnızca tek bir değer alır.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lPushx**

Mevcut bir listenin başına bir değer ekler, liste mevcut değilse işlem geçersiz ve 0 döner. Key liste türünde değilse false döner.

Dönen değer lPushx komutu çalıştırıldıktan sonra listenin uzunluğudur.

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

Belirtilen aralıktaki liste öğelerini döndürür. 0 listesinin ilk öğesini, 1 liste ikinci öğesini temsil eder. Negatif indekseler de kullanılabilir, -1 liste son öğesini, -2 liste sondan bir önceki öğeyi temsil eder.

Belirtilen aralıktaki öğeleri içeren bir dizi döner. Eğer key liste türünde değilse false döner.

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

Parametre olarak verilen COUNT değerine göre liste içindeki VALUE ile eşleşen öğeleri kaldırır.

COUNT değeri şu şekilde olabilir: 

*   count > 0 : Başından itibaren VALUE ile eşleşen öğeleri kaldırır, miktar COUNT olur.
*   count < 0 : Sondan başlayarak VALUE ile eşleşen öğeleri kaldırır, miktar COUNT'un mutlak değeridir.
*   count = 0 : Tüm VALUE değerlerini kaldırır.

Kaldırılan öğelerin sayısını döner. Liste mevcut değilse 0 döner. Liste değilse false döner.

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

İndeksine göre bir öğenin değerini ayarlar.

Başarılı olduğunda true döner, indeks parametresi aralık dışında ise veya boş bir liste için LSET kullanıldığında false döner.

```php
$redis->lSet('key1', 0, 'X');
```

## **lTrim**

Bir listeyi belirli bir aralıkta tutarak diğer öğeleri siler.

0 indis listenin ilk öğesini, 1 listenin ikinci öğesini temsil eder. Negatif indekseler de kullanılabilir, -1 liste son öğesini, -2 liste sondan bir önceki öğeyi temsil eder.

Başarılı olduğunda true, başarısız olduğunda false döner.

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

Listenin son öğesini kaldırır ve kaldırılan öğeyi döndürür.

Eğer liste mevcut değilse, null döner.

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush**

Listenin son öğesini kaldırır ve bu öğeyi başka bir listeye ekler, ardından öğeyi döndürür.

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

Bir veya daha fazla değeri listenin sonuna (sağına) ekler ve eklenen listenin uzunluğunu döndürür.

Eğer liste mevcut değilse, boş bir liste oluşturulur ve RPUSH işlemi gerçekleştirilir. Listenin mevcut olması ancak liste türünde olmaması durumunda false döner.

**Not:** Redis 2.4 sürümünden önceki RPUSH komutları yalnızca tek bir değer alır.

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX**

Mevcut bir listenin sonuna bir değer ekler ve listenin uzunluğunu döndürür. Eğer liste mevcut değilse işlem geçersiz ve 0 döner. Listenin mevcut olması ancak liste türünde olmaması durumunda false döner.

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen**

Liste uzunluğunu döndürür. Eğer liste key mevcut değilse, key boş bir liste olarak kabul edilir ve 0 döner. Listenin liste türünde olmaması durumunda false döner.

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

Bir veya daha fazla üye öğeyi kümelere ekler; kümede zaten varsa yok sayılır.

Eğer küme key mevcut değilse, yalnızca eklenen öğelerden oluşan bir küme oluşturulur.

Küme key'in küme türünde olmaması durumunda false döner.

**Not:** Redis 2.4 sürümünden önceki SADD yalnızca tek bir üye değer alır.

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

Kümedeki öğelerin sayısını döndürür. Eğer küme key mevcut değilse 0 döner.

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

Birinci kümenin diğer kümelere göre farkını döndürür, yani birinci kümede bulunan öğeleri döndürür. Mevcut olmayan küme key'leri boş küme olarak kabul edilir.

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

Belirtilen kümeler arasındaki farkları belirtilen kümeye kaydeder. Belirtilen küme key zaten varsa üzerine yazar.

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

Verilen tüm kümelerin kesişimini döndürür. Mevcut olmayan küme key'leri boş küme olarak kabul edilir. Verilen küme içinde en az bir boş küme varsa sonuç da boş küme olur.

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

Belirtilen kümeler arasındaki kesişimi belirtilen bir kümede depolar ve depolanan kesişim kümesinin eleman sayısını döndürür. Belirtilen küme zaten varsa üzerine yazar.

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

Üye elemanın kümeye ait olup olmadığını belirler.

Eğer üye eleman kümenin bir üyesiyse 1 döner. Eğer üye eleman kümenin bir üyesi değilse veya anahtar (key) mevcut değilse 0 döner. Eğer anahtar (key) bir küme türünde değilse false döner.

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## **sMembers**

Kümedeki tüm üyeleri döndürür. Mevcut olmayan bir küme anahtarı boş bir küme olarak kabul edilir.

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## **sMove**

Belirtilen üye elemanı source kümesinden destination kümesine taşır.

SMOVE atomik bir işlemdir.

Eğer source küme mevcut değilse veya belirtilen üye elemanı içermiyorsa, SMOVE komutu herhangi bir işlem yapmaz, sadece 0 döner. Aksi takdirde, üye eleman source kümesinden kaldırılır ve destination kümesine eklenir.

Destination kümesi zaten üye elemanı içeriyorsa, SMOVE komutu sadece source kümesinden üye elemanını siler.

Eğer source veya destination bir küme türünde değilse false döner.

```php
$redis->sMove('key1', 'key2', 'member13');
```

## **sPop**

Belirtilen anahtara sahip kümeden bir veya daha fazla rastgele elemanı kaldırır ve kaldırılan elemanı döndürür.

Küme mevcut değilse veya boş bir küme ise null döner.

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

Redis Srandmember komutu, bir kümenin rastgele bir elemanını döndürmek için kullanılır.

Redis 2.6 sürümünden itibaren, Srandmember komutu isteğe bağlı count parametresini kabul eder:

* count pozitif bir sayıysa ve küme temel sayısından küçükse, komut count adet eleman içeren bir dizi döndürür ve dizideki elemanlar birbirinden farklıdır. Eğer count küme temel sayısından büyük veya eşitse, tüm küme döner.
* count negatif bir sayıysa, komut elemanları tekrar edebilen count mutlak değeri uzunluğunda bir dizi döndürür.

Bu işlem SPOP komutuna benzer, ancak SPOP kümeden rastgele bir elemanı kaldırır ve döndürürken, Srandmember sadece rastgele elemanı döndürür, küme üzerinde herhangi bir değişiklik yapmaz.

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

Kümeden bir veya daha fazla üye elemanı kaldırır, mevcut olmayan üye elemanlar göz ardı edilir.

Başarıyla kaldırılan elemanların sayısını döndürür, göz ardı edilen elemanlar dâhil değildir.

Anahtar (key) küme türünde değilse false döner.

Redis 2.4 sürümünden önce, SREM sadece tek bir üye değeri kabul ederdi.

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

Belirtilen kümelerin birleşimini döndürür. Mevcut olmayan küme anahtarları boş bir küme olarak kabul edilir.

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

Belirtilen kümelerin birleşimini belirtilen destination kümesinde depolar, ve eleman sayısını döndürür. Eğer destination mevcutsa üzerine yazar.

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
