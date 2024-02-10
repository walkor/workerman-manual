# workerman/redis-queue

Fila de mensagens baseada em Redis, suportando processamento de mensagens com atraso.

## Endereço do projeto:
https://github.com/walkor/redis-queue

## Instalação:
```composer require workerman/redis-queue```

## Exemplo
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // Inscrever-se
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // Inscrever-se
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // Enviar mensagens para a fila em intervalos regulares
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

Cria uma instância

  * `$address`  Similar a `redis://ip:6379`, deve começar com redis.

  * `$options`  Inclui as seguintes opções:
    * `auth`: Informações de autenticação, padrão ''
    * `db`: db, padrão 0
    * `max_attempts`: Número de tentativas de repetição após falha de consumo, padrão 5
    * `retry_seconds`: Intervalo de tempo de repetição, em segundos. Padrão 5

> Falha de consumo significa que a operação de negócios lança uma exceção `Exception` ou `Error`. Após a falha de consumo, a mensagem será colocada na fila de atraso para tentativa de repetição, o número de tentativas de repetição é controlado por `max_attempts`, e o intervalo de repetição é controlado pela combinação de `retry_seconds` e `max_attempts`. Por exemplo, se `max_attempts` for 5 e `retry_seconds` for 10, o intervalo de repetição da primeira repetição é `1*10` segundos, o intervalo de repetição da segunda repetição é `2*10` segundos, o intervalo de repetição da terceira repetição é `3*10` segundos, e assim por diante até a quinta repetição. Se o número de tentativas de repetição ultrapassar o configurado em `max_attempts`, a mensagem será colocada na fila de falhas com a chave `{redis-queue}-failed` (anteriormente era `{redis-queue-failed}` até a versão 1.0.5)

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

Envia uma mensagem para a fila

* `$queue` Nome da fila, tipo `String`
* `$data` Mensagem específica enviada, pode ser uma matriz ou uma string, tipo `Mixed`
* `$dely` Tempo de atraso para consumo, em segundos, padrão 0, tipo `Int`
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

Inscreve-se em uma ou várias filas

* `$queue` Nome da fila, pode ser uma string ou um array contendo vários nomes de fila
* `$callback` Função de retorno, com o formato `function (Mixed $data)`, onde `$data` é o mesmo que `$data` em `send($queue, $data)`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

Cancela a inscrição em uma ou várias filas

* `$queue` Nome da fila ou um array contendo vários nomes de fila

-------------------------------------------------------

## Enviando mensagens para a fila em um ambiente não workerman
Às vezes, alguns projetos são executados em um ambiente apache ou php-fpm, onde não é possível usar o projeto workerman/redis-queue. Nesses casos, você pode usar a seguinte função para enviar mensagens
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //antes da versão 1.0.5 era redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';//antes da versão 1.0.5 era redis-queue-delayed
    
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
Onde, o parâmetro `$redis` é uma instância do redis. Por exemplo, o uso da extensão do redis seria semelhante ao seguinte:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data = ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```
