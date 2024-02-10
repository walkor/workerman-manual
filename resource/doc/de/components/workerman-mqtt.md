# webman/mqtt

MQTT ist ein Publish/Subscribe-Nachrichtenübertragungsprotokoll mit Client-Server-Architektur und hat sich zu einem wichtigen Bestandteil des Internets der Dinge entwickelt. Sein Designkonzept ist leichtgewichtig, offen, einfach und standardisiert und daher einfach zu implementieren. Diese Merkmale machen es in vielen Szenarien zu einer guten Wahl, insbesondere für eingeschränkte Umgebungen wie Maschine-zu-Maschine-Kommunikation (M2M) und das Internet der Dinge (IoT).

webman\mqtt ist eine auf Workerman basierende asynchrone MQTT-Client-Bibliothek, die für den Empfang oder das Senden von MQTT-Protokollnachrichten verwendet werden kann. Es unterstützt `QoS 0`, `QoS 1`, `QoS 2` und die Versionen `MQTT` `3.1`, `3.1.1`, `5`.

## Projektadresse
https://github.com/walkor/mqtt

## Installation
```composer require workerman/mqtt```

## Beispiel
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
Zum Starten in der Befehlszeile eingeben: ```php subscribe.php start```

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
Zum Starten in der Befehlszeile eingeben: ```php publish.php start```

## webman\mqtt\Client-Schnittstelle
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

Erstellt eine MQTT-Client-Instanz.

  * `$address` Adresse des MQTT-Servers im Format 'mqtt://test.mosquitto.org:1883'. 

  * `$options` Optionen-Array für den Client, das folgende Optionen enthalten kann:
    * `keepalive`: Zeitintervall, in dem der Client regelmäßige Herzschläge an den Server sendet, standardmäßig 50 Sekunden, mit 0 wird kein Herzschlag aktiviert
    * `client_id`: Client-ID, wenn nicht festgelegt, wird standardmäßig ```"workerman-mqtt-client-".mt_rand()```
    * `protocol_name`: Name des Protokolls, `MQTT`(Version 3.1.1) oder `MQIsdp`(Version 3.1) standardmäßig `MQTT`
    * `protocol_level`: Protokollversion, 4, wenn `protocol_name` `MQTT` ist, und 3, wenn `protocol_name` `MQIsdp` ist
    * `clean_session`: Clean-Sitzung, standardmäßig `true`. Wenn auf `false` gesetzt, können Offline-Nachrichten mit `QoS 1` und `QoS 2` empfangen werden
    * `reconnect_period`: Zeitspanne für die erneute Verbindung, standardmäßig 1 Sekunde, 0 bedeutet keine erneute Verbindung
    * `connect_timeout`: Verbindungs-Timeout für MQTT, standardmäßig 30 Sekunden
    * `username`: Benutzername, optional
    * `password`: Passwort, optional
    * `will`: Testament-Nachricht, die der Broker automatisch an andere Clients sendet, wenn der Client getrennt ist. Das Format ist:
      * `topic`: Thema
      * `content`: Inhalt
      * `qos`: `QoS`-Level
      * `retain`: `retain`-Flag
    * `resubscribe` : Beim erneuten Verbinden nach einem Verbindungsfehler die zuvor abonnierten Themen erneut abonnieren, standardmäßig `true`
    * `bindto`: Legt fest, mit welcher IP-Adresse und welchen Port der Client eine Verbindung zum Broker herstellen soll, standardmäßig ''
    * `ssl`: SSL-Option, standardmäßig `false`. Wenn auf `true` gesetzt, wird eine SSL-Verbindung hergestellt. Zusätzlich kann ein Array für den SSL-Kontext übergeben werden, um lokale Zertifikate zu konfigurieren. Der SSL-Kontext ist dem folgenden Link zu entnehmen: https://php.net/manual/en/context.ssl.php
    * `debug`: Aktiviert den Debug-Modus, welcher detaillierte Informationen zur Kommunikation mit dem Broker ausgibt, standardmäßig `false`

-------------------------------------------------------

<a name="connect"></a>
### connect()

Stellt eine Verbindung zum Broker her.

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

Veröffentlicht eine Nachricht zu einem bestimmten Thema

* `$topic` Thema
* `$content` Nachricht
* `$options` Options-Array, einschließlich
  * `qos` `QoS`-Level, standardmäßig `0`
  * `retain` Behalten-Flag, standardmäßig `false`
  * `dup` Wiederholungsflag, standardmäßig `false`
* `$callback` - `function (\Exception $exception = null)`, (Diese Einstellungen werden nicht für QoS 0 unterstützt) wird ausgelöst, wenn ein Fehler auftritt oder die Veröffentlichung erfolgreich ist. `$exception` ist das Ausnahmeobjekt, das `null` ist, wenn kein Fehler aufgetreten ist, und dasselbe gilt für die folgenden.

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

Abonniert ein oder mehrere Themen

* `$topic` ist ein String (ein Thema abonnieren) oder ein Array (mehrere Themen abonnieren),
wenn mehrere Themen abonniert werden, ist `$topic` ein Array mit Themen als Schlüssel und QoS als Wert, z.B. `array('topic1'=> 0, 'topic2'=> 1)`
* `$options` Abonnement-Options-Array, einschließlich der folgenden Einstellungen:
  * `qos` `QoS`-Level, standardmäßig `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])`
  Callback-Funktion, die ausgelöst wird, wenn das Abonnieren erfolgreich ist oder ein Fehler auftritt
  * `exception` Ausnahmeobjekt, das `null` ist, wenn kein Fehler auftritt, und dasselbe gilt für die folgenden
  * `granted` Array mit Ergebnissen des Abonnements, ähnlich wie `array('topic' => 'qos', 'topic' => 'qos')` wobei:
    * `topic` das abonnierte Thema ist
    * `qos` das empfangene `QoS`-Level des Brokers

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

Beendet das Abonnement

* `$topic` ist ein String oder ein Array von Strings, ähnlich wie `array('topic1', 'topic2')`
* `$callback` - `function (\Exception $e = null)`, wird ausgelöst, wenn erfolgreich oder fehlerhaft

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

Trennt die Verbindung zum Broker ordnungsgemäß, `DISCONNECT`-Nachricht wird an den Broker gesendet.

-------------------------------------------------------

<a name="close"></a>
### close()

Trennt die Verbindung zum Broker gewaltsam, es wird keine `DISCONNECT`-Nachricht an den Broker gesendet.

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)
Wird ausgelöst, wenn die Verbindung zum Broker hergestellt ist. Zu diesem Zeitpunkt wurde bereits die `CONNACK`-Nachricht des Brokers empfangen.

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)
`function (topic, message, packet) {}`

Wird ausgelöst, wenn der Client eine Publish-Nachricht empfängt
* `$topic` das empfangene Thema
* `$content` der empfangene Nachrichteninhalt
* `$mqtt` MQTT-Clientinstanz.

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)
Wird ausgelöst, wenn ein Verbindungsfehler auftritt

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()
Wird ausgelöst, wenn die Verbindung geschlossen wird, unabhängig davon, ob die Client oder der Server die Verbindung trennt.
