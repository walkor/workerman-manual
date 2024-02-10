# workerman/mqtt
MQTT è un protocollo di trasporto dei messaggi di tipo publish/subscribe con architettura client/server, diventato una parte importante dell'Internet delle cose. Il suo design è leggero, aperto, semplice, standardizzato e facile da implementare. Queste caratteristiche lo rendono una scelta eccellente per molti scenari, specialmente per comunicazioni in ambienti limitati come la comunicazione tra macchine (M2M) e l'ambiente dell'Internet delle cose (IoT).

workerman\mqtt è una libreria client MQTT asincrona basata su Workerman, utilizzata per ricevere o inviare messaggi con il protocollo MQTT. Supporta i livelli di servizio 'QoS 0', 'QoS 1' e 'QoS 2', e le versioni 'MQTT' '3.1', '3.1.1' e '5'.

# Indirizzo del progetto
https://github.com/walkor/mqtt

# Installazione 
```php
composer require workerman/mqtt
```

# Esempio
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
Eseguire da terminale con il comando  ```php subscribe.php start``` per avviare

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

Eseguire da terminale con il comando ```php publish.php start``` per avviare.

## Interfaccia di workerman\mqtt\Client

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

Crea un'istanza del client MQTT.

  * `$address` indirizzo del server MQTT, nel formato 'mqtt://test.mosquitto.org:1883'. 

  * `$options` array di opzioni per il client, che può includere le seguenti opzioni:
    * `keepalive`: intervallo di invio dei battiti cardiaci dal client al server, di default 50 secondi, impostare a 0 indica che i battiti cardiaci non sono attivi
    * `client_id`: id del client, se non impostato di default è ```"workerman-mqtt-client-".mt_rand()```
    * `protocol_name`: nome del protocollo, 'MQTT'(versione 3.1.1) o 'MQIsdp'(versione 3.1), di default è 'MQTT'
    * `protocol_level`: livello del protocollo, se `protocol_name` è 'MQTT' il valore è `4`, se è 'MQIsdp' il valore è `3`
    * `clean_session`: sessione pulita, di default è `true`. Impostando a `false` consente di ricevere messaggi offline di livello `QoS 1` e `QoS 2`
    * `reconnect_period`: intervallo di ricongiunzione, di default `1` secondo, `0` indica nessuna riconnessione
    * `connect_timeout`: timeout della connessione MQTT, di default `30` secondi
    * `username`: nome utente, opzionale
    * `password`: password, opzionale
    * `will`: messaggio di testamento, quando il client si disconnette, il broker invierà automaticamente un messaggio di testamento ad altri client. Formato:
      * `topic`: argomento
      * `content`: contenuto
      * `qos`: livello `QoS`
      * `retain`: flag `retain`
    * `resubscribe` : quando la connessione si interrompe in modo anomalo e viene ristabilita, ri-sottoscrivi i topic precedenti, di default è true
    * `bindto` specifica l'indirizzo IP e la porta locale da usare per connettersi al broker, di default è ''
    * `ssl` opzioni ssl, di default è `false`, se impostato a `true`, la connessione avviene tramite ssl. Supporta anche l'inserimento di un array di contesti ssl per configurare certificati locali, etc., il contesto ssl si riferisce a https://php.net/manual/en/context.ssl.php
    * `debug` abilita o disabilita la modalità di debug, la modalità debug può generare informazioni dettagliate sulla comunicazione con il broker, di default è `false`

-------------------------------------------------------

<a name="connect"></a>
### connect()

Connette al broker.

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

Pubblica un messaggio su un determinato topic.

* `$topic` argomento
* `$content` messaggio
* `$options` array di opzioni, comprensivo di
  * `qos` livello `QoS`, di default `0`
  * `retain` flag di conservazione, di default `false`
  * `dup` flag di duplicazione, di default `false`
* `$callback` - `function (\Exception $exception = null)` (non supportato per QoS 0) triggerato in caso di errore o pubblicazione riuscita, `$exception` è l'oggetto eccezione, in assenza di errori `$exception` è `null`, il medesimo per le seguenti.

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

Sottoscrive un singolo argomento o più argomenti.

* `$topic` è una stringa (per sottoscrivere un singolo argomento) o un array (per sottoscrivere più argomenti),
quando si sottoscrivono più argomenti, `$topic` è un array in cui l'argomento è la chiave e il livello `QoS` è il valore, ad esempio `array('argomento1' => 0, 'argomento2' => 1)`
* `$options` array di opzioni di sottoscrizione, che include le seguenti impostazioni:
  * `qos` livello `QoS`, di default `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])`
  funzione di callback, triggerata in caso di successo o fallimento della sottoscrizione
  * `exception` oggetto eccezione, in assenza di errori è `null`
  * `granted` array di risultati della sottoscrizione, simile a `array('argomento' => 'qos', 'argomento' => 'qos')` in cui:
    * `argomento` è l'argomento sottoscritto
    * `qos` livello `QoS` accettato dal broker

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

Cancella la sottoscrizione di un singolo o più argomenti.

* `$topic` è una stringa o un array di stringhe, ad esempio `array('argomento1', 'argomento2')`
* `$callback` - `function (\Exception $e = null)`, triggerato in caso di successo o fallimento

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

Disconnette dal broker, invia il messaggio `DISCONNECT` al broker.

-------------------------------------------------------

<a name="close"></a>
### close()

Forza la disconnessione dal broker, non invia il messaggio `DISCONNECT` al broker.

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)
Triggerato quando la connessione con il broker è stabilita. A questo punto è stato ricevuto il messaggio `CONNACK` dal broker.

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)
`function (topic, message, packet) {}`

Triggerato quando il cliente riceve un messaggio di tipo publish
* `$topic` argomento ricevuto
* `$content` contenuto del messaggio ricevuto
* `$mqtt` istanza del client MQTT

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)
Triggerato in caso di errore di connessione

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()
Triggerato quando la connessione viene chiusa, sia per la chiusura volontaria del client che per la chiusura del server.
