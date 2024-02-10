# workerman/redis-queue

Coda di messaggi basata su Redis con supporto per la gestione ritardata dei messaggi.

## Indirizzo del progetto:
https://github.com/walkor/redis-queue

## Installazione:
```bash
composer require workerman/redis-queue
```

## Esempio
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // Iscrizione
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // Iscrizione
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // Invio periodico di messaggi alla coda
    Timer::add(1, function()use($client){
        $client->send('user-1', ['some', 'data']);
    });
};

Worker::runAll();
```

## API
  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#send"><code>Client::<b>send()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

Crea un'istanza

  * `$address`  Simile a `redis://ip:6379`, deve iniziare con redis. 

  * `$options`  Include le seguenti opzioni:
    * `auth`: Informazioni di autenticazione, di default ''
    * `db`: db, di default 0
    * `max_attempts`: Numero di tentativi di ripetizione dopo il fallimento del consumo, di default 5
    * `retry_seconds`: Intervallo di tempo per il ripetizione, in secondi. Di default 5

> Il fallimento del consumo si riferisce al lancio di un'eccezione `Exception` o `Error` nell'attività. Dopo il fallimento, il messaggio viene messo in una coda di ritardo in attesa di ripetizione. Il numero di tentativi di ripetizione è controllato da `max_attempts`, mentre l'intervallo di ripetizione è controllato da `retry_seconds` e `max_attempts`. Ad esempio, se `max_attempts` è 5 e `retry_seconds` è 10, l'intervallo di ripetizione per il primo tentativo è `1*10` secondi, per il secondo tentativo è `2*10` secondi, per il terzo tentativo è `3*10` secondi e così via fino a 5 tentativi di ripetizione. Se il numero di tentativi di ripetizione supera il valore di `max_attempts`, il messaggio viene inserito nella coda dei fallimenti con la chiave `{redis-queue}-failed` (prima della versione 1.0.5 era `-redis-queue-failed`)

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

Invia un messaggio alla coda

* `$queue` Nome della coda, di tipo `String`
* `$data` Messaggio specifico inviato, può essere un array o una stringa, di tipo `Mixed`
* `$dely` Tempo di ritardo del consumo, in secondi, di default 0, di tipo `Int`
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

Sottoscrive una o più code

* `$queue` Nome della coda, può essere una stringa o un array di nomi di code
* `$callback` Funzione di richiamo con formato `function (Mixed $data)`, dove `$data` è lo stesso di `$data` in `send($queue, $data)`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

Annulla la sottoscrizione

* `$queue` Nome della coda o un array di nomi di code

-------------------------------------------------------

## Invio di messaggi alla coda in un ambiente non workerman
A volte, alcuni progetti vengono eseguiti in un ambiente Apache o PHP-FPM e non è possibile utilizzare il progetto workerman/redis-queue. Puoi fare riferimento alla seguente funzione per implementare l'invio
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //prima della versione 1.0.5 era `-redis-queue-waiting`
    $queue_delay = '{redis-queue}-delayed';//prima della versione 1.0.5 era `-redis-queue-delayed`
    
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```
Dove il parametro `$redis` è un'istanza di redis. Ad esempio, l'uso dell'estensione di redis è simile a quanto segue:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```
