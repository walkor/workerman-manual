# workerman/mqtt
MQTT เป็นโปรโตคอลการส่งข่าวสารแบบการต่อ/สมัครที่มีโครงสร้างของลูกค้าและเซิร์ฟเวอร์ ซึ่งได้กลายเป็นส่วนสำคัญของอินเทอร์เน็ตของสิ่งของ การออกแบบของมันมุ่งไปที่ความเบาบาง ความเปิดเป้า ความง่าย และความเป็นมาตรฐาน ทำให้มันเป็นทางเลือกที่ดีสำหรับหลายๆ สถานการณ์ โดยเฉพาะอย่างยิ่งสำหรับสภาพแวดล้อมที่ จำกัด เช่น การสื่อสารระหว่างเครื่องกับเครื่อง (M2M) และสภาพแวดล้อมของอินเทอร์เน็ตของสิ่งของ (IoT) 

workerman\mqtt เป็นไลบรารีของลูกค้า MQTT แบบไม่ระบบเชื่อมต่อ ที่มีการออกแบบขึ้นบน workerman ซึ่งสามารถใช้สำหรับการรับหรือส่งข้อความโปรโตคอล MQTT รองรับ `QoS 0`  `QoS 1`  `QoS 2` รองรับเวอร์ชันของ `MQTT` `3.1` `3.1.1` `5`

# ลิงก์โปรเจค
https://github.com/walkor/mqtt

# การติดตั้ง
```bash
composer require workerman/mqtt
```

# ตัวอย่าง
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
ใช้คำสั่งใน Command line  ```php subscribe.php start``` เพื่อเริ่มต้น

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
ใช้คำสั่งใน Command line ```php publish.php start``` เพื่อเริ่มต้น 
## อินเทอร์เฟซของ workerman\mqtt\Client

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

สร้างตัวอย่างของไคลเอ็นต์ mqtt

  * `$address` ที่อยู่เซิร์ฟเวอร์ mqtt รูปแบบเช่น 'mqtt://test.mosquitto.org:1883'.

  * `$options` อาร์เรย์ตัวเลือกของไคลเอ็นต์ สามารถตั้งค่าต่อไปนี้ได้:
    * `keepalive`: ช่วงเวลาที่ไคลเอ็นต์ส่งดวงใจไปยังเซิร์ฟเวอร์ ค่าเริ่มต้นคือ 50 วินาที ตั้งค่าเป็น 0 แทนถ้าไม่ต้องการเปิดใช้งานดวงใจ
    * `client_id`: ไอดีของไคลเอ็นต์ ถ้าไม่ได้ตั้งค่าจะเป็น ```"workerman-mqtt-client-".mt_rand()```
    * `protocol_name`: ชื่อโพรโตคอล `MQTT` (เวอร์ชัน 3.1.1) หรือ `MQIsdp` (เวอร์ชัน 3.1) ค่าเริ่มต้นคือ `MQTT`
    * `protocol_level`: ระดับโพรโตคอล เมื่อ `protocol_name` เป็น `MQTT` ค่าเป็น `4` เมื่อ `protocol_name` เป็น `MQIsdp` ค่าเป็น `3`
    * `clean_session`: เซสชั่นที่ดูแล้ว ค่าเริ่มต้นคือ `true` การตั้งค่าเป็น `false` จะเป็นการรับข้อมูลการตั้งค่า QoS 1 และ QoS 2 ของข้อความแบบออฟไลน์
    * `reconnect_period`: ช่วงเวลาสำหรับการเชื่อมต่อใหม่ ค่าเริ่มต้นคือ `1` วินาที ค่า `0` แทนการเชื่อมต่อแบบไม่ต้องการ
    * `connect_timeout`: ช่วงเวลาสำหรับการเชื่อมต่อ mqtt ค่าเริ่มต้นคือ `30` วินาที
    * `username`: ชื่อผู้ใช้ ตัวเลือก
    * `password`: รหัสผ่าน ตัวเลือก
    * `will`: ข้อความอวิโฆษณีเมื่อไคลเอ็นต์ตัดการเชื่อมต่อ โฆษณาฮิ่งะที่เซิร์ฟเวอร์จะส่งข้อความอวิโฆษณีให้ไคลเอ็นต์อื่น ๆ รูปแบบดังนี้:
      * `topic`: หัวข้อ
      * `content`: เนื้อหา
      * `qos`: ระดับ `QoS`
      * `retain`: ติดสัญลัฏแปลงไปเก็บ
    * `resubscribe`: เมื่อการเชื่อมต่อกับสิ่งที่ผิดพลาดและเชื่อมต่อใหม่ เช่นในการตั้งค่าใหม่เป็น `true`
    * `bindto` ใช้เพื่อระบุไอพีและพอร์ตท้องถิ่นในการเชื่อมต่อไปยังโบรกเกอร์ ค่าเริ่มต้นคือ ''
    * `ssl` ตัวเลือกของ ssl ค่าเริ่มต้นคือ `false` ถ้าตั้งค่าเป็น `true` จะเชื่อมต่อโดยใช้ ssl ในขณะเดียวกันสนับสนุนข้อมูลของการต่อตาม ssl ที่สามารถกำหนดให้กับใบรับรองภายใน ssl และอื่น ๆ ssl ติดต่อดูข้อมูลที่เกี่ยวกับ https://php.net/manual/en/context.ssl.php
    * `debug` จะเปิดโหมดดีบักเอารห์ โหมดดีบักเอารห์สามารถเอาออกข้อมูลการสื่อสารระหว่างกับโบรกเกอร์ค่าเริ่มต้นคือ `false`

-------------------------------------------------------

<a name="connect"></a>
### connect()

เชื่อมต่อโบรกเกอร์

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

เผยแพร่ข้อความไปยังหัวข้อหนึ่ง

* `$topic` หัวข้อ
* `$content` ข้อความ
* `$options` อาร์เรย์ตัวเลือก, รวมถึง
  * `qos` ระดับ `QoS` เริ่มต้น `0`
  * `retain` ติดสัญลัฏแปลงไปเก็บ เริ่มต้น `false`
  * `dup` ซ้ำซ้อนโลโก้เริ่มต้น `false`
* `$callback` - `function (\Exception $exception = null)`  (ไม่สนับสนุนการตั้งค่า QoS เป็น 0) เมื่อเกิดข้อผิดพลาดหรือเมื่อเผยแพร่สำเร็จจะกระตุ้น `$exception` เป็นอ็อบเจ็กต กฎอื่น ๆ

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

สมัครบัญชีหัวข้อหรือหัวข้อหลาย ๆ

* `$topic` เป็นสตริง (สมัครบัญชีหัวข้อหนึ่ง) หรืออาร์เรย์ (สมัครบัญชีหัวข้อหลาย ๆ) เมื่อสมัครบัญชีหัวข้อหลาย ๆ `$topic` เป็นหัวข้อตาม คีย์ และ `QoS` เป็น ค่าของอาร์เรย์ เช่น `array('topic1'=> 0, 'topic2'=> 1)`
* `$options` อาร์เรย์ของการตั้งค่าการสมัครบัญชีรวมถึงการตั้งค่าต่อไปนี้:
  * `qos` ระดับ `QoS` เริ่มต้น `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])` ฟังก์ชันตอบสนองเมื่อสมัครบัญชีสำเร็จหรือเกิดข้อผิดพลาด 
  * `exception` อ็อบเจ็กตข้อผิดพลาดเมื่อไม่มีข้อผิดพลาดมันคือ `null` กตติดต่อหรือกระตุ้นฤทธิ์การสมัครบัญชี ในบางเส้น

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

ยกเลิกการสมัครบัญชี

* `$topic` เป็นสตริงหรืออาร์เรย์สตริง เช่น `array('topic1', 'topic2')`
* `$callback` - `function (\Exception $e = null)`  กระตุ้นเมื่อสำเร็จหรือไม่สำเร็จ

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

ตัดการติดต่อกับโบรกเกอร์ไปตามปกติ `DISCONNECT` ข้อผูกติดต่อจะถูกส่งไปยังโโปรแกรมจัดการสื่อสาร

-------------------------------------------------------

<a name="close"></a>
### close()

ตัดการติดต่อกับโบรกเกอร์เป็นการบังคับ ๆ ไม่ส่ง `DISCONNECT` ข้อผูกติดต่อไปยังโโปรแกรมจัดการสื่อสาร

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)

เมื่อติดต่อโบรกเกอร์สร้างเสร็จ โดยนี้ได้รับ `CONNACK` ข้อผูกติดต่อ

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)

`function (topic, message, packet) {}`

เมื่อไคลเอ็นต์รับข้อความจากไคลเอ็นต์
* `$topic` หัวข้อที่ไคลเอ็นต์รับมา
* `$content` เนื้อหาข้อความที่ไคลเอ็นต์รับมา
* `$mqtt` ตัวอย่างไคลเอ็นต์ mqtt.

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)

เมื่อเกิดข้อผิดพลาดการเชื่อมต่อ

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()

เมื่อการเชื่อมต่อปิดไปตามปกติไม่ว่าจะเป็นไคลเอ็นต์ปิดเองหรือเซิร์ฟเวอร์ปิดจะกระตุ้น `onClose`
