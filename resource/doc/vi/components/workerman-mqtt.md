# workerman/mqtt
MQTT là một giao thức truyền tin nhắn theo mô hình phát hành/đăng ký giữa máy khách và máy chủ, đã trở thành một phần quan trọng của Internet of Things. Thiết kế của nó nhẹ nhàng, mở, đơn giản, chuẩn mực và dễ triển khai. Những đặc điểm này khiến nó trở thành lựa chọn tốt cho nhiều tình huống, đặc biệt là đối với môi trường có hạn như việc giao tiếp giữa máy và máy (M2M) cũng như môi trường Internet of Things (IoT).

workerman\mqtt là một thư viện người dùng mqtt không đồng bộ dựa trên workerman, có thể được sử dụng để nhận hoặc gửi tin nhắn theo giao thức MQTT. Hỗ trợ `QoS 0`, `QoS 1`, `QoS 2`. Hỗ trợ các phiên bản `MQTT` `3.1`, `3.1.1`, `5`.


# Địa chỉ dự án
https://github.com/walkor/mqtt

# Cài đặt
```
composer require workerman/mqtt
```

# Ví dụ
**subscribe.php**
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $mqtt = new Workerman\Mqtt\Client('mqtt://test.mosquitto.org:1883');
    $mqtt->onConnect = function($mqtt) {
        $mqtt->subscribe('test');
    };
    $mqtt->onMessage = function($topic, $content){
        var_dump($topic, $content);
    };
    $mqtt->connect();
};
Worker::runAll();
```
Chạy dòng lệnh  ```php subscribe.php start``` để bắt đầu

**publish.php**
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $mqtt = new Workerman\Mqtt\Client('mqtt://test.mosquitto.org:1883');
    $mqtt->onConnect = function($mqtt) {
       $mqtt->publish('test', 'hello workerman mqtt');
    };
    $mqtt->connect();
};
Worker::runAll();
```

Chạy dòng lệnh ```php publish.php start``` để bắt đầu.

## workerman\mqtt\Client interface

  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#connect"><code>Client::<b>connect()</b></code></a>
  * <a href="#publish"><code>Client::<b>publish()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>
  * <a href="#disconnect"><code>Client::<b>disconnect()</b></code></a>
  * <a href="#close"><code>Client::<b>close()</b></code></a>
  * <a href="#onConnect"><code>callback <b>onConnect</b></code></a>
  * <a href="#onMessage"><code>callback <b>onMessage</b></code></a>
  * <a href="#onError"><code>callback <b>onError</b></code></a>
  * <a href="#onClose"><code>callback <b>onClose</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

Tạo một phiên bản người dùng mqtt.

  * `$address` Địa chỉ máy chủ mqtt, định dạng tương tự 'mqtt://test.mosquitto.org:1883'. 

  * `$options` Mảng tùy chọn cho người dùng, có thể thiết lập các tùy chọn sau:
    * `keepalive`: Thời gian gửi tin nhắn mạch hở từ máy khách đến máy chủ, mặc định là 50 giây, thiết lập thành 0 để tắt tính năng gửi tin nhắn mạch hở
    * `client_id`: ID máy khách, nếu không thiết lập, mặc định là ```"workerman-mqtt-client-".mt_rand()```
    * `protocol_name`: Tên giao thức, `MQTT`(phiên bản 3.1.1) hoặc `MQIsdp`(phiên bản 3.1), mặc định là `MQTT`
    * `protocol_level`: Mức giao thức, nếu `protocol_name` là `MQTT`, giá trị là `4`, nếu `protocol_name` là `MQIsdp`, giá trị là `3`
    * `clean_session`: Phiên làm sạch, mặc định là `true`. Thiết lập thành `false` để nhận tin nhắn ngoại tuyến cấp `QoS 1` và `QoS 2`
    * `reconnect_period`: Khoảng thời gian tái kết nối, mặc định là `1` giây, `0` để không kết nối lại
    * `connect_timeout`: Thời gian kết nối mqtt vượt quá thời gian chờ, mặc định là `30` giây
    * `username`: Tên người dùng, tùy chọn
    * `password`: Mật khẩu, tùy chọn
    * `will`: Tin nhắn lưu lại, khi máy khách ngắt kết nối, Broker sẽ tự động gửi tin nhắn lưu lại đến các máy khách khác. Định dạng:
      * `topic`: Chủ đề
      * `content`: Nội dung
      * `qos`: Cấp `QoS`
      * `retain`: Cờ `retain`
    * `resubscribe` : Khi kết nối bị ngắt kết nối và kết nối lại, liệu có đăng ký lại chủ đề trước đó không, mặc định là true
    * `bindto` Dùng để chỉ định kết nối từ IP và cổng nào của máy chủ tới Broker, giá trị mặc định là ''
    * `ssl` Tùy chọn ssl, mặc định là `false`, nếu thiết lập thành `true`, kết nối bằng ssl. Đồng thời hỗ trợ truyền mảng ngữ cảnh ssl, dùng để thiết lập chứng chỉ cục bộ, v.v., tham khảo ngữ cảnh ssl tại https://php.net/manual/en/context.ssl.php
    * `debug` Có bật chế độ gỡ lỗi hay không, chế độ gỡ lỗi có thể hiển thị thông tin chi tiết về giao tiếp với Broker, mặc định là `false`

-------------------------------------------------------

<a name="connect"></a>
### connect()

Kết nối với Broker

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

Đăng một tin nhắn tới một chủ đề nào đó

* `$topic` Chủ đề
* `$content` Tin nhắn
* `$options` Mảng tùy chọn, bao gồm
  * `qos` cấp `QoS`, mặc định là `0`
  * `retain` cờ lưu lại, mặc định là `false`
  * `dup` Cờ gửi lại, mặc định là `false`
* `$callback` - `function (\Exception $exception = null)` (Không hỗ trợ khi cấp `QoS` là 0), khi có lỗi xảy ra hoặc đăng tin thành công, `$exception` là đối tượng ngoại lệ, khi không có lỗi xảy ra, `$exception` là `null`, tương tự như vậy.

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

Đăng ký một chủ đề hoặc nhiều chủ đề

* `$topic` là một chuỗi (đăng ký một chủ đề) hoặc một mảng (đăng ký nhiều chủ đề), 
khi đăng ký nhiều chủ đề, `$topic` là mảng chủ đề làm khóa và giá trị là cấp `QoS`, ví dụ như array('topic1'=> 0, 'topic2'=> 1)
* `$options` Mảng tùy chọn đăng ký, bao gồm các thiết lập sau:
  * `qos` cấp `QoS`, mặc định là `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])`
  Hàm gọi lại, khi đăng ký thành công hoặc có lỗi xảy ra
  * `exception` Đối tượng ngoại lệ, khi không có lỗi xảy ra là `null`, tương tự như vậy.
  * `granted` Mảng kết quả đăng ký, tương tự như `array('topic' => 'qos', 'topic' => 'qos')`, trong đó:
    * `topic` là chủ đề đăng ký
    * `qos` Mức `QoS` mà Broker chấp nhận

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

Hủy đăng ký

* `$topic` là một chuỗi hoặc mảng chuỗi, tương tự như `array('topic1', 'topic2')`
* `$callback` - `function (\Exception $e = null)`, hàm gọi lại khi thành công hoặc thất bại

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

Ngắt kết nối với Broker một cách bình thường, gửi `DISCONNECT` bản tin đến cho Broker.

-------------------------------------------------------

<a name="close"></a>
### close()

Ngắt kết nối mạnh với Broker, sẽ không gửi `DISCONNECT` bản tin đến cho Broker.

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)
Khi kết nối với Broker hoàn tất, sẽ gọi hàm này. Lúc này đã nhận được bản tin `CONNACK` từ Broker.

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)
`function (topic, message, packet) {}`

Khi máy khách nhận được bản tin Publish.
* `$topic` Chủ đề nhận được
* `$content` Nội dung tin nhắn nhận được
* `$mqtt` Phiên bản máy khách mqtt.

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)
Khi có lỗi xảy ra trong quá trình kết nối

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()
Khi kết nối đóng, dù là máy khách đóng kết nối tự ý hoặc máy chủ đóng kết nối, hàm này đều sẽ được gọi.
