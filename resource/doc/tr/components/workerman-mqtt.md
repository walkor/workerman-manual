# workerman/mqtt

MQTT, istemci-sunucu mimarisi üzerine kurulu bir yayın/abone modeli mesaj iletim protokolüdür ve nesnelerin interneti'nin önemli bir parçası haline gelmiştir. Tasarımı hafif, açık, basit, standart ve uygulanabilir olma düşüncesine dayanır. Bu özellikler, pek çok senaryo için ideal bir seçim olmasını sağlar, özellikle de makine ile makine iletişimi (M2M) ve nesnelerin interneti (IoT) gibi sınırlı ortamlar için.

workerman\mqtt, workerman üzerinde asenkron MQTT istemci kütüphanesidir ve MQTT protokolü mesajlarını almak veya göndermek için kullanılabilir. `QoS 0`, `QoS 1`, `QoS 2`'yi destekler. `MQTT` `3.1`, `3.1.1`, `5` sürümlerini destekler.

# Proje adresi
https://github.com/walkor/mqtt

# Kurulum
```composer require workerman/mqtt```

# Örnek
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
Komut satırında çalıştır ```php subscribe.php start``` başlatmak için

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
Komut satırında çalıştır ```php publish.php start``` başlatmak için.

## workerman\mqtt\Client arayüzü

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

Bir MQTT istemci örneği oluşturur.

  * `$address` MQTT sunucu adresi, formatı 'mqtt://test.mosquitto.org:1883' gibi.
  * `$options` istemci seçenekleri dizisi, aşağıdaki seçenekleri ayarlayabilir:
    * `keepalive`: İstemci tarafından sunucuya gönderilen kalp atışı aralığı, varsayılan 50 saniye, 0'a ayarlanırsa kalp atışı kullanılmaz
    * `client_id`: İstemci kimliği, ayarlanmadıysa varsayılan olarak ```"workerman-mqtt-client-".mt_rand()```
    * `protocol_name`: Protokol adı, `MQTT` (3.1.1 sürümü) veya `MQIsdp` (3.1 sürümü), varsayılan olarak `MQTT`
    * `protocol_level`: Protokol seviyesi, `protocol_name` `MQTT` olduğunda `4` değeri, `protocol_name` `MQIsdp` olduğunda `3` değeridir
    * `clean_session`: Temiz oturum, varsayılan olarak `true`. `false` olarak ayarlanırsa, `QoS 1` ve `QoS 2` düzeyinde offline mesajlar alınabilir
    * `reconnect_period`: Yeniden bağlanma süresi aralığı, varsayılan `1` saniye, `0` bağlantının yeniden kurulmaması anlamına gelir
    * `connect_timeout`: MQTT bağlantı zaman aşımı süresi, varsayılan `30` saniye
    * `username`: Kullanıcı adı, isteğe bağlı
    * `password`: Şifre, isteğe bağlı
    * `will`: Veda mesajı, istemci kesildiğinde Broker otomatik olarak diğer istemcilere veda mesajı gönderir. Format:
      * `topic`: Konu
      * `content`: İçerik
      * `qos`: `QoS` seviyesi
      * `retain`: `retain` işareti
    * `resubscribe`: Bağlantı kesildiğinde ve yeniden bağlandığında, önceki konuları yeniden abone edip etmeyeceği, varsayılan olarak `true`
    * `bindto`: Broker'a hangi IP ve porttan bağlantı başlatılacağını belirtmek için kullanılır, varsayılan değer `''`
    * `ssl`: ssl seçeneği, varsayılan olarak `false`, `true` olarak ayarlanırsa ssl ile bağlantı kurar. Aynı zamanda ssl bağlamak için ssl bağlamak için yerel sertifika vb. ayarlamak için ssl context dizisi geçilebilir, ssl context, https://php.net/manual/en/context.ssl.php adresinden incelenebilir
    * `debug`: hata ayıklama modunun açılıp kapatılması, hata ayıklama modu ile Broker ile iletişim ayrıntıları çıkartılır, varsayılan olarak `false`

-------------------------------------------------------

<a name="connect"></a>
### connect()

Broker ile bağlantı kurar

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

Bir konuya bir mesaj yayınlar

* `$topic` konu
* `$content` mesaj
* `$options` seçenekler dizisi, içerir
  * `qos` `QoS` seviyesi, varsayılan `0`
  * `retain` retain işareti, varsayılan `false`
  * `dup` tekrar gönderme bayrağı, varsayılan `false`
* `$callback` - `function (\Exception $exception = null)` (QoS 0 için bu ayar desteklenmez), bir hata meydana geldiğinde veya yayın başarılı olduğunda tetiklenir, `$exception` hata nesnesidir, hata meydana gelmediğinde `$exception` `null`'dır, aynı durum geçerlidir.

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

Bir konuyu veya birden fazla konuyu abone olur

* `$topic` bir dize (bir konu abonelik) veya bir dizi (çoklu konu abonelik)dir,
birden fazla konu abone olunacaksa, `$topic` anahtar olacak konu, değeri `QoS` olacak bir dizi, örneğin `array('konu1' => 0, 'konu2' => 1)` olmalıdır
* `$options` abone seçenekleri dizisi, aşağıdakileri içerir:
  * `qos` `QoS` seviyesi, varsayılan `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])`
  geri çağırma fonksiyonu, abonelik başarılı veya bir hata meydana geldiğinde tetiklenir
  * `exception` hata nesnesi, hata olmadığında `null`'dır, diğerleri için aynı geçerlidir
  * `granted` abonelik sonuç dizisi, `array('konu' => 'qos', 'konu' => 'qos')` gibi, burada:
    * `konu` abone olunan konu
    * `qos` Broker'ın kabul ettiği `QoS` seviyesi

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

Aboneliği iptal eder

* `$topic` bir dize veya dize dizisi, `array('konu1', 'konu2')` gibi
* `$callback` - `function (\Exception $e = null)`, başarılı veya başarısız olduğunda tetiklenen geri çağırma

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

Broker ile normal bağlantıyı keser, `DISCONNECT` paketi Broker'a gönderilir.

-------------------------------------------------------

<a name="close"></a>
### close()

Broker'la zorla bağlantıyı keser, Broker'a `DISCONNECT` paketi gönderilmez.

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)
Broker ile bağlantı kurulduğunda tetiklenir. Bu noktada Broker'ın `CONNACK` paketi alınmış olacaktır.

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)
`function (topic, message, packet) {}`

İstemci bir Yayın paketi aldığında tetiklenir
* `$topic` alınan konu
* `$content` alınan mesaj içeriği
* `$mqtt` MQTT istemci örneği

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)
Bağlantıda bir hata oluştuğunda tetiklenir

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()
Bağlantı kapatıldığında tetiklenir, istemci tarafından kapatılmış olsun ya da sunucu tarafından kapatılmış olsun`onClose` tetiklenecektir
