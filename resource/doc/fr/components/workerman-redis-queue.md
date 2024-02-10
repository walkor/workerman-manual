# workerman/redis-queue

File d'attente de messages basée sur Redis avec prise en charge du traitement différé des messages.

## Adresse du projet :
https://github.com/walkor/redis-queue

## Installation :
```shell
composer require workerman/redis-queue
```

## Exemple
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // Abonnement
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // Abonnement
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // Envoi périodique de messages à la file d'attente
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

Crée une instance

  * `$address`  de type `redis://ip:6379`, doit commencer par redis. 

  * `$options`  comprend les options suivantes :
    * `auth` : informations d'authentification, par défaut ''
    * `db` : base de données, par défaut 0
    * `max_attempts` : nombre de tentatives de réessai après un échec de consommation, par défaut 5
    * `retry_seconds` : intervalle de temps entre les réessais, en secondes. Par défaut 5

> Un échec de consommation se produit lorsqu'une exception `Exception` ou une `Error` est levée dans le traitement du message. Après un échec de consommation, le message est placé dans une file d'attente de traitement différé pour une nouvelle tentative. Le nombre de tentatives de réessai est contrôlé par `max_attempts`, et l'intervalle de réessai est contrôlé par `retry_seconds` et `max_attempts`. Par exemple, si `max_attempts` est 5 et `retry_seconds` est 10, le temps d'attente pour la première tentative de réessai est `1*10` secondes, pour la deuxième tentative c'est `2*10` secondes, pour la troisième c'est `3*10` secondes, et ainsi de suite jusqu'à la cinquième tentative. Si le nombre de tentatives de réessai dépasse la valeur de `max_attempts`, le message est placé dans la file d'attente des échecs avec la clé `{redis-queue}-failed` (avant la version 1.0.5, elle était `{redis-queue-failed}`).

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

Envoie un message à la file d'attente

* `$queue` : nom de la file d'attente, de type `String`
* `$data` : message spécifique à publier, peut être un tableau ou une chaîne de caractères, de type `Mixed`
* `$dely` : délai de consommation différée, en secondes, par défaut 0, de type `Int`
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

Abonne à une file d'attente ou à plusieurs files d'attente

* `$queue` : nom de la file d'attente, peut être une chaîne de caractères ou un tableau contenant plusieurs noms de files d'attente
* `$callback` : fonction de rappel, au format `function (Mixed $data)`, où `$data` représente `$data` du `send($queue, $data)`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

Annule l'abonnement

* `$queue` : nom de la file d'attente ou un tableau contenant plusieurs noms de files d'attente

-------------------------------------------------------

## Envoi de messages à la file d'attente dans un environnement non Workerman
Parfois, certains projets s'exécutent dans un environnement apache ou php-fpm et ne peuvent pas utiliser le projet workerman/redis-queue. Vous pouvez vous référer à la fonction suivante pour mettre en œuvre l'envoi :
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //avant la version 1.0.5, c'était redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';//avant la version 1.0.5, c'était redis-queue-delayed
    
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
Ici, le paramètre `$redis` est une instance de Redis. Par exemple, l'utilisation de l'extension Redis est similaire à ce qui suit :
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```
