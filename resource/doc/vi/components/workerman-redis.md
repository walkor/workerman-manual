# workman-redis

## Giới thiệu

workeman/redis là một thành phần redis không đồng bộ dựa trên workerman.

> **Chú ý**
> Mục tiêu chính của dự án này là thực hiện đăng ký không đồng bộ redis (subscribe, pSubscribe)
> Vì redis đủ nhanh, nếu không cần đăng ký không đồng bộ psubscribe subscribe, không cần sử dụng khách hàng không đồng bộ này, sử dụng phần mở rộng redis sẽ có hiệu suất tốt hơn

## Cài đặt:
```
composer require workerman/redis
```

## Sử dụng callback

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

## Sử dụng coroutine

> **Chú ý**
> Sử dụng coroutine yêu cầu workerman>=5.0, workerman/redis>=2.0.0 và cài đặt composer require revolt/event-loop ^1.0.0

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

Khi không thiết lập hàm callback, khách hàng sẽ trả kết quả yêu cầu không đồng bộ theo cách đồng bộ, quá trình yêu cầu không chặn quá trình hiện tại, có nghĩa là có thể xử lý đồng thời các yêu cầu.

> **Chú ý**
> psubscribe subscribe Không hỗ trợ coroutine

# Tài liệu
**Giải thích**

**Trong cách gọi callback, hàm callback thông thường có 2 tham số ($result, $redis), `$result` là kết quả, `$redis` là thể hiện redis. Ví dụ:**
```
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// Setup hàm callback để kiểm tra kết quả gọi set
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// Hàm callback là tùy chọn, ở đây hàm callback được bỏ qua
$redis->set('key1', 'value1');
// Hàm callback có thể lồng nhau
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **Kết nối**
```php
use Workerman\Redis\Client;
// Bỏ qua callback
$redis = new Client('redis://127.0.0.1:6379');
// Có callback
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // Thiết lập thời gian chờ kết nối 10 giây, nếu không thiết lập, mặc định là 5 giây
], function ($success, $redis) {
    // Callback kết quả kết nối
    if (!$success) echo $redis->error();
});
```

## **auth**
```php
// Xác thực mật khẩu
$redis->auth('password', function ($result) {
    
});
// Xác thực tên người dùng và mật khẩu
$redis->auth('username', 'password', function ($result) {

});
```

## **pSubscribe**

Đăng ký một hoặc nhiều kênh phù hợp với mẫu được cung cấn.

Mỗi mẫu sử dụng dấu * như dấu phù hợp, ví dụ it\* sẽ phù hợp với tất cả các kênh bắt đầu bằng it (vd: it.news, it.blog, it.tweets và như vậy). news.\* sẽ phù hợp với tất cả các kênh bắt đầu bằng news. (vd: news.it, news.global.today và như vậy).

Lưu ý: Hàm callback pSubscribe được gọi với 4 tham số ($pattern, $channel, $message, $redis)

Khi thể hiện $redis gọi pSubscribe hoặc subscribe, các phương thức khác của thể hiện hiện tại sẽ bị bỏ qua.
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

Được sử dụng để đăng ký các thông tin từ một hoặc nhiều kênh đã cho.

Lưu ý: Hàm callback subscribe được gọi với 3 tham số ($channel, $message, $redis)

Khi thể hiện $redis gọi pSubscribe hoặc subscribe, các phương thức khác của thể hiện hiện tại sẽ bị bỏ qua.
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

Được sử dụng để gửi thông tin đến kênh chỉ định

Trả về số lượng người đăng ký nhận thông tin.
```php
$redis2->publish('news', 'news content');
```

## **select**
```php
// Bỏ qua callback
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // Tham số select phải là số, nên ở đây $result là false
    var_dump($result, $redis->error());
});
```

## **get**

Lệnh được sử dụng để lấy giá trị của khóa đã chỉ định. Nếu khóa không tồn tại, trả về NULL. Nếu giá trị khóa không phải là kiểu chuỗi, trả về false.
```php
$redis->get('key', function($result) {
     // Nếu khóa không tồn tại thì trả về NULL, nếu có lỗi thì trả về false
    var_dump($result);
});
```

## **set**

Được sử dụng để thiết lập giá trị cho khóa cụ thể. Nếu khóa đã tồn tại giá trị khác, SET sẽ ghi đè giá trị cũ và bỏ qua kiểu dữ liệu.
```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// Tham số thứ ba có thể truyền thời gian hết hạn, hết hạn sau 10 giây
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **setEx, pSetEx**

Thiết lập giá trị và thời gian hết hạn cho khóa định sẵn. Nếu khóa đã tồn tại, lệnh SETEX sẽ thay thế giá trị cũ.
```php
// Lưu ý tham số thứ hai truyền thời gian hết hạn, tính bằng giây
$redis->setEx('key', 3600, 'value'); 
// pSetEx tính bằng mili giây
$redis->pSetEx('key', 3600, 'value'); 
```

## **del**

Sử dụng để xóa khóa đã tồn tại, kết quả trả về là số, đại diện cho số lượng khóa đã xóa (khóa không tồn tại không tính)
```php
// Xóa một khóa
$redis->del('key');
// Xóa nhiều khóa
$redis->del(['key', 'key1', 'key2']);
```

## **setNx**

(Lệnh **SET**if**N**ot e**X**ists) được sử dụng khi khóa cụ thể không tồn tại, để thiết lập giá trị cho khóa.
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

Lệnh được sử dụng để kiểm tra xem khóa đã cho có tồn tại hay không. Kết quả trả về là số, đại diện cho số lượng khóa tồn tại.
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

Thay đổi giá trị của khóa theo cách tăng lên 1/giá trị chỉ định. Nếu khóa không tồn tại, giá trị của khóa sẽ được khởi tạo bằng 0 trước khi thực hiện lệnh incr/incrBy.
Nếu giá trị chứa kiểu dữ liệu sai, hoặc giá trị kiểu chuỗi không thể biểu diễn dưới dạng số, lệnh sẽ trả về false.
Khi thành công, sẽ trả về giá trị sau khi thay đổi.
```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **incrByFloat**

Thay đổi giá trị của khóa theo cách thêm vào giá trị số thập phân được chỉ định. Nếu khóa không tồn tại, lệnh INCRBYFLOAT sẽ thiết lập cho khóa giá trị là 0 trước khi thực hiện lệnh cộng. Nếu giá trị chứa kiểu dữ liệu sai, hoặc giá trị kiểu chuỗi không thể biểu diễn dưới dạng số, lệnh sẽ trả về false. Khi thành công, sẽ trả về giá trị sau khi thay đổi.
```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## **decr, decrBy**

Lệnh giảm giá trị của khóa theo cách giảm đi 1/giá trị chỉ định. Nếu khóa không tồn tại, giá trị của khóa sẽ được khởi tạo bằng 0 trước khi thực thi lệnh decr/decrBy.
Nếu giá trị chứa kiểu dữ liệu sai, hoặc giá trị kiểu chuỗi không thể biểu diễn dưới dạng số, lệnh sẽ trả về false.Khi thành công, sẽ trả về giá trị sau khi thay đổi.
```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

Trả về tất cả (một hoặc nhiều) giá trị của khóa đã cho. Nếu trong các khóa đã cho, có một hoặc nhiều khóa không tồn tại, thì sẽ trả về NULL.
```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```
## **getSet**

Dùng để đặt giá trị cho key chỉ định và trả về giá trị cũ của key đó.

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

Trả về ngẫu nhiên một key từ cơ sở dữ liệu hiện tại.
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    });
})
```

## **move**

Di chuyển key trong cơ sở dữ liệu hiện tại đến cơ sở dữ liệu db chỉ định.
```php
$redis->select(0);	// chuyển sang DB 0
$redis->set('x', '42');	// viết 42 vào x
$redis->move('x', 1, function ($result) { 	// di chuyển đến DB 1
    var_dump($result); // 1
});  
$redis->select(1);	// chuyển sang DB 1
$redis->get('x', function ($result) {
    var_dump($result); // '42'
});
```

## **rename**

Đổi tên của key, nếu key không tồn tại thì trả về false.
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
});
```

## **renameNx**

Đổi tên của key khi key mới không tồn tại.
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
});
```

## **expire**

Đặt thời gian hết hạn cho key, sau khi hết hạn key sẽ không còn sử dụng được nữa. Đơn vị là giây. Trả về 1 nếu thành công, trả về 0 nếu key không tồn tại, trả về false nếu có lỗi.
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

Lệnh để tìm tất cả các key phù hợp với mẫu pattern cho trước.
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
});
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
});
```

## **type**

Trả về loại giá trị mà key đó lưu trữ. Kết quả trả về là một trong các chuỗi sau: string, set, list, zset, hash hoặc none, trong đó none biểu thị key không tồn tại.
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
});
```

## **append**

Nếu key đã tồn tại và là một chuỗi, lệnh APPEND sẽ thêm value vào cuối giá trị ban đầu của key và trả về độ dài của chuỗi. 

Nếu key không tồn tại, APPEND sẽ đơn giản chỉ đặt key với value, tương tự như lệnh SET key value và trả về độ dài của chuỗi.

Nếu key tồn tại nhưng không phải chuỗi, trả về false.
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

Nhận ra một chuỗi con của chuỗi được lưu trữ trong key chỉ định. Phạm vi cắt chuỗi được quyết định bởi hai offset start và end (bao gồm start và end). Nếu key không tồn tại, trả về chuỗi rỗng. Nếu key không phải là kiểu chuỗi, trả về false.
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

Ghi đè giá trị chuỗi trữ trong key chỉ định với chuỗi được chỉ định, vị trí ghi đè bắt đầu từ offset. Nếu key không tồn tại, đặt key thành chuỗi được chỉ định. Nếu key không phải là chuỗi, trả về false.

Kết quả trả về là độ dài của chuỗi sau khi sửa đổi.
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

Nhận độ dài của giá trị chuỗi của key chỉ định. Khi key không chứa giá trị chuỗi, trả về false.
```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
}); 
```

## **getBit**

Đọc bit cụ thể ở vị trí offset trong giá trị chuỗi được lưu trữ của key đã chỉ định.
```php
$redis->set('key', "\x7f"); // this is 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
}); 
```

## **setBit**

Thiết lập hoặc xóa bit cụ thể ở vị trí offset trong giá trị chuỗi được lưu trữ của key đã chỉ định. Giá trị trả về là 0 hoặc 1, là giá trị trước khi sửa đổi.
```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2a = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
});
```

## **bitOp**

Thực hiện phép toán theo bit (AND, OR, XOR, NOT) trên nhiều key (bao gồm giá trị chuỗi) và lưu kết quả vào key đích.

Kết quả trả về là kích thước của chuỗi lưu trong key đích, tức là bằng kích thước của chuỗi dài nhất đầu vào.
```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
});
```

## **bitCount**

Tính số bit được đặt thành 1 (đếm dân số) trong chuỗi.

Mặc định, nó sẽ kiểm tra tất cả các byte trong chuỗi. Chỉ có thể chỉ định thêm tham số *start* và *end* trong khoảng để thực hiện đếm.

Tương tự như lệnh GETRANGE, start và end có thể chứa giá trị âm để chỉ mục byte từ cuối chuỗi, trong đó -1 là byte cuối cùng, -2 là byte thứ hai từ cuối cùng, vân vân.

Kết quả trả về số lượng bit có giá trị 1 trong chuỗi.

Nếu key không tồn tại, nó sẽ được coi là chuỗi rỗng và lệnh sẽ trả về 0.
```php
$redis->set('key', 'hello');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
});
$redis->bitCount( 'key', function ($result) {
    var_dump($result); // 21
});
```

## **sort**

Lệnh sort có thể sắp xếp các phần tử trong list, set và sorted set.

Dạng: `sort($key, $options, $callback);`

Nơi options có thể bao gồm các key và giá trị tùy chọn sau:
~~~
$options = [
     'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // hoặc một mảng các pattern
    'sort' => 'asc', // hoặc 'desc'
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

Trả về thời gian hết hạn còn lại của key theo đơn vị giây hoặc mili giây.

Nếu key không có thời hạn, trả về -1. Nếu key không tồn tại, trả về -2.
```php
$redis->set('key', 'value', 10);
// Theo giây
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// Theo mili giây
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// Key không tồn tại
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```

## **persist**

Loại bỏ thời hạn hết hạn của key nhất định, làm cho key không bị hết hạn. 

Nếu loại bỏ thành công, trả về 1, nếu key không tồn tại hoặc không có thời gian hết hạn, trả về 0, nếu có lỗi xảy ra, trả về false.
```php
$redis->persist('key');
```

## **mSet, mSetNx**

Đặt nhiều cặp key-value trong một lệnh nguyên tử. mSetNx chỉ trả về 1 khi tất cả các key được đặt. 

Trả về 1 nếu thành công, trả về 0 nếu thất bại, trả về false nếu xảy ra lỗi.
```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

Đặt giá trị cho trường trong hash. 

Nếu trường là một trường mới và giá trị được thiết lập thành công, trả về 1. Nếu trường đã tồn tại trong hash và giá trị cũ đã bị ghi đè bằng giá trị mới, trả về 0.
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

Đặt giá trị cho trường trong hash nếu trường chưa tồn tại.

Nếu hash không tồn tại, sẽ tạo một hash mới và thực hiện HSET. 

Nếu trường đã tồn tại trong hash, lệnh sẽ không thực hiện gì. 

Nếu key không tồn tại, sẽ tạo một bảng băm mới và thực hiện lệnh HSETNX.
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

Trả về giá trị của trường cụ thể trong hash. 

Nếu trường hoặc key không tồn tại, trả về null.
```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```
## **hLen**

Được sử dụng để lấy số lượng trường trong bảng băm.

Trả về 0 khi khóa không tồn tại.

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

Lệnh được sử dụng để xóa một hoặc nhiều trường chỉ định trong bảng băm key và các trường không tồn tại sẽ bị bỏ qua.

Trả về số lượng trường bị xóa thành công, không bao gồm các trường bị bỏ qua. Nếu key không phải là hash, nó sẽ trả về false.

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

Lấy tất cả các trường trong bảng băm dưới dạng mảng.

Nếu key không tồn tại, nó sẽ trả về một mảng rỗng. Nếu key không phải là hash, nó sẽ trả về false.

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

Trả về tất cả các giá trị của trường trong bảng băm dưới dạng mảng.

Nếu key không tồn tại, nó sẽ trả về một mảng rỗng. Nếu key không phải là hash, nó sẽ trả về false.

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

Trả về tất cả các trường và giá trị trong bảng băm dưới dạng mảng liên kết.

Nếu key không tồn tại, nó sẽ trả về một mảng rỗng. Nếu key không phải là hash, nó sẽ trả về false.

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
Trả về
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

## **hExists**

Kiểm tra xem trường chỉ định trong bảng băm có tồn tại hay không. Trả về 1 nếu tồn tại, 0 nếu trường không tồn tại hoặc key không tồn tại, và false nếu có lỗi xảy ra.

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## **hIncrBy**

Dùng để tăng giá trị trường chỉ định trong bảng băm lên một số lượng cụ thể.

Số lượng cũng có thể là số âm, tương đương với việc thực hiện phép trừ. Nếu khóa bảng băm không tồn tại, một bảng băm mới sẽ được tạo và thực hiện lệnh HINCRBY. Nếu trường chỉ định không tồn tại, giá trị của trường sẽ được khởi tạo là 0 trước khi thực hiện lệnh.

Thực hiện lệnh HINCRBY trên trường chứa giá trị chuỗi sẽ trả về false.

Giá trị thực hiện lệnh bị giới hạn trong phạm vi của 64 bit số nguyên có dấu.

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

Tương tự như hIncrBy, chỉ khác là số lượng cũng có thể là số thực.

## **hMSet**

Đồng thời đặt nhiều cặp field-value vào bảng băm.

Lệnh này sẽ ghi đè lên các trường đã tồn tại trong bảng băm. Nếu bảng băm không tồn tại, sẽ tạo một bảng băm trống và thực hiện lệnh HMSET.

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

Trả về các giá trị của các trường đã cho trong bảng băm dưới dạng mảng liên kết.

Nếu trường chỉ định không tồn tại trong bảng băm, giá trị tương ứng sẽ là null. Nếu key không phải là hash, nó sẽ trả về false.

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
Kết quả:
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```

## **blPop, brPop**

Di chuyển và lấy phần tử đầu tiên/cuối cùng của danh sách. Nếu danh sách không có phần tử, nó sẽ chờ đợi cho đến khi có phần tử có thể lấy hoặc quá thời gian chờ đợi.

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

Lấy phần tử cuối cùng của danh sách và đưa vào đầu danh sách khác; nếu danh sách không có phần tử, nó sẽ chờ đợi cho đến khi có phần tử có thể lấy hoặc sau quá thời gian chờ đợi. Nếu quá thời gian chờ đợi, nó sẽ trả về null.

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

Lấy phần tử trong danh sách theo chỉ mục. Cũng có thể sử dụng chỉ mục âm, -1 biểu thị phần tử cuối cùng của danh sách, -2 biểu thị phần tử thứ hai từ cuối, và cứ thế.

Nếu chỉ mục chỉ định không nằm trong phạm vi của danh sách, nó sẽ trả về null. Nếu key không phải là danh sách, nó sẽ trả về false.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

Chèn phần tử trước hoặc sau một phần tử trong danh sách. Không thực hiện bất kỳ hành động nào nếu phần tử chỉ định không tồn tại trong danh sách hoặc danh sách không tồn tại.

Nếu key không phải là danh sách, nó sẽ trả về false.

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

Xóa và trả về phần tử đầu tiên của danh sách.

Trả về null khi danh sách key không tồn tại.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```

## **lPush**

Chèn một hoặc nhiều giá trị vào đầu danh sách. Nếu key không tồn tại, một danh sách mới sẽ được tạo và thực hiện lệnh LPUSH. Nếu key tồn tại nhưng không phải là danh sách, nó sẽ trả về false.

**Chú ý:** lệnh LPUSH trước phiên bản Redis 2.4 chỉ chấp nhận một giá trị duy nhất.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lPushx**

Chèn một giá trị vào đầu danh sách tồn tại; nếu danh sách không tồn tại, lệnh không thực hiện và trả về 0. Nếu key không phải là danh sách, nó sẽ trả về false.

Giá trị trả về sau khi thực hiện lệnh lPushx là độ dài của danh sách.

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

Trả về một mảng chứa các phần tử của danh sách nằm trong một khoảng chỉ định. Chỉ mục 0 biểu thị phần tử đầu tiên, 1 biểu thị phần tử thứ hai, và cứ thế.

Nếu key không phải là danh sách, nó sẽ trả về false.

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

Dùng số lượng COUNT để xóa các phần tử giống với giá trị VALUE trong danh sách.

Giá trị của COUNT có thể là:

*   count > 0 : Từ phần tử đầu tiên đến phần tử cuối cùng trong danh sách, xóa COUNT phần tử giống với VALUE.
*   count < 0 : Từ phần tử cuối cùng đến phần tử đầu tiên trong danh sách, xóa COUNT phần tử giống với VALUE.
*   count = 0 : Xóa tất cả các phần tử giống với VALUE.

Trả về số lượng phần tử đã bị xóa. Trả về 0 khi danh sách không tồn tại. Trả về false khi không phải là danh sách.

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

Thay đổi giá trị của phần tử trong danh sách theo chỉ mục.

Trả về true khi thành công, trả về false nếu chỉ mục vượt quá phạm vi hoặc danh sách rỗng.

```php
$redis->lSet('key1', 0, 'X');
```
## **lTrim**

lTrim được sử dụng để cắt tỉa một danh sách (list), chỉ giữ lại các phần tử trong khoảng được chỉ định và loại bỏ các phần tử không nằm trong khoảng đó.

Chỉ số 0 biểu thị phần tử đầu tiên trong danh sách, 1 đại diện cho phần tử thứ hai và cứ thế. Bạn cũng có thể sử dụng chỉ số âm, -1 biểu thị phần tử cuối cùng, -2 biểu thị phần tử trước cuối và cứ thế.

Khi thành công sẽ trả về true, khi thất bại trả về false.

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

Sử dụng để loại bỏ phần tử cuối cùng của danh sách và trả về giá trị của phần tử đó.

Khi danh sách không tồn tại, sẽ trả về null.

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush**

Loại bỏ phần tử cuối cùng của danh sách và thêm phần tử đó vào danh sách khác rồi trả về giá trị.

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

Chèn một hoặc nhiều giá trị vào cuối danh sách và trả về độ dài mới của danh sách.

Nếu danh sách không tồn tại, một danh sách trống sẽ được tạo ra và thực hiện thao tác RPUSH. Khi danh sách tồn tại nhưng không phải là kiểu danh sách sẽ trả về false.

**Chú ý:** Trong phiên bản Redis 2.4 trở về trước, lệnh RPUSH chỉ chấp nhận một giá trị duy nhất.

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX**

Chèn một giá trị vào cuối danh sách đã tồn tại và trả về độ dài của danh sách. Nếu danh sách không tồn tại, thao tác sẽ không có hiệu lực và trả về 0. Khi danh sách tồn tại nhưng không phải là kiểu danh sách sẽ trả về false.

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen**

Trả về độ dài của danh sách. Nếu danh sách key không tồn tại, key sẽ được coi là danh sách rỗng và trả về 0. Nếu key không phải là kiểu danh sách, sẽ trả về false.

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

Thêm một hoặc nhiều phần tử thành viên vào tập hợp, những phần tử thành viên đã tồn tại sẽ bị bỏ qua.

Nếu tập hợp key không tồn tại, tạo mới một tập hợp chỉ chứa các phần tử được thêm. Khi key không phải là kiểu tập hợp, sẽ trả về false.

**Chú ý:** Trong phiên bản Redis 2.4 trở về trước, lệnh SADD chỉ chấp nhận một thành viên duy nhất.

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

Trả về số lượng phần tử trong tập hợp. Khi tập hợp key không tồn tại, trả về 0.

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

Trả về sự khác biệt giữa tập hợp đầu tiên với các tập hợp khác, cũng có thể hiểu là các phần tử duy nhất trong tập hợp đầu tiên. Các tập hợp không tồn tại sẽ được coi là tập hợp rỗng.

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

Lưu sự khác biệt giữa các tập hợp đã cho vào tập hợp được chỉ định. Nếu tập hợp đã được chỉ định tồn tại, nó sẽ bị ghi đè.

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

Trả về giao của tất cả các tập hợp đã cho. Các tập hợp không tồn tại sẽ được coi là tập hợp rỗng. Khi một trong các tập hợp đã cho là tập hợp rỗng, kết quả cũng là tập hợp rỗng.

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

Lưu giao của các tập hợp đã cho vào tập hợp được chỉ định và trả về số lượng phần tử trong tập hợp giao đó. Nếu tập hợp đã cho đã tồn tại, nó sẽ bị ghi đè.

```php
$redi

s->sAdd('key1', 'val1');
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

Kiểm tra xem phần tử thành viên có trong tập hợp không. Nếu thành viên là thành viên của tập hợp, trả về 1. Nếu thành viên không phải là thành viên của tập hợp hoặc tập hợp không tồn tại, trả về 0. Nếu key không phải là kiểu tập hợp, trả về false.

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## **sMembers**

Trả về tất cả các thành viên của tập hợp. Khi tập hợp không tồn tại, trả về tập hợp rỗng.

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## **sMove**

Di chuyển phần tử thành viên chỉ định từ tập hợp nguồn sang tập hợp đích.

SMOVE là một hoạt động nguyên tử.

Nếu tập hợp nguồn không tồn tại hoặc không chứa thành viên chỉ định, lệnh SMOVE sẽ không thực hiện bất kỳ hoạt động nào, chỉ trả về 0. Ngược lại, thành viên từ tập hợp nguồn sẽ bị loại bỏ và thêm vào tập hợp đích. Khi tập hợp đích đã chứa thành viên, lệnh SMOVE chỉ đơn giản là loại bỏ thành viên từ tập hợp nguồn.

Khi tập hợp nguồn hoặc đích không phải là kiểu tập hợp, trả về false.

```php
$redis->sMove('key1', 'key2', 'member13');
```

## **sPop**

Xóa một hoặc nhiều phần tử ngẫu nhiên từ tập hợp và trả về phần tử đã xóa.

Khi tập hợp không tồn tại hoặc là tập hợp rỗng, trả về null.

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

Lệnh Srandmember của Redis được sử dụng để trả về một phần tử ngẫu nhiên từ tập hợp.

Từ phiên bản Redis 2.6 trở lên, lệnh Srandmember chấp nhận tham số đếm tùy chọn:

* Nếu đếm là số dương và nhỏ hơn số phần tử trong tập hợp, lệnh trả về một mảng chứa đúng số phần tử, mỗi phần tử là duy nhất. Nếu đếm lớn hơn hoặc bằng số phần tử trong tập hợp, lệnh trả về toàn bộ tập hợp.
* Nếu đếm là số âm, lệnh trả về một mảng, các phần tử có thể lặp lại nhiều lần và chiều dài mảng là giá trị tuyệt đối của đếm.

Thao tác này tương tự như SPOP, tuy nhiên SPOP loại bỏ phần tử ngẫu nhiên từ tập hợp và trả về, trong khi Srandmember chỉ trả về phần tử ngẫu nhiên mà không thay đổi tập hợp.

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

Xóa một hoặc nhiều phần tử thành viên khỏi tập hợp, các phần tử thành viên không tồn tại sẽ bị bỏ qua.

Trả về số lượng phần tử đã được xóa thành công, không bao gồm các phần tử bị bỏ qua.

Nếu `key` không phải là một kiểu tập hợp, sẽ trả về false.

Trước phiên bản Redis 2.4, SREM chỉ chấp nhận một giá trị thành viên duy nhất.

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

Lệnh trả về hợp của các tập hợp được chỉ định. Các tập hợp không tồn tại được coi là tập hợp rỗng.

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

Lưu trữ hợp của các tập hợp được chỉ định vào tập hợp đích đã cho và trả về số lượng phần tử. Nếu tập hợp đích đã tồn tại, nó sẽ bị ghi đè.

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
