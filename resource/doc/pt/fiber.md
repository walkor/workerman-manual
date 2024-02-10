# Fiber Coroutines
O workerman suporta [Fiber Coroutines (Fibras)](https://www.php.net/manual/zh/language.fibers.php) a partir da versão 5.0.0.

> **Observação**
> A funcionalidade de Fibra requer PHP >= 8.1 e a instalação de `composer require revolt/event-loop ^1.0.0`.

### Introdução
Fiber é uma corrotina (fibra) incorporada no PHP, que pode interromper o código PHP e retomar sua execução quando necessário. Sua principal vantagem é permitir que os desenvolvedores escrevam código assíncrono não bloqueante de forma síncrona, o que aumenta significativamente a manutenibilidade do código.

### Exemplo
A diferença entre programação de corrotina e retorno de chamada assíncrono pode ser ilustrada por exemplo. Suponha que haja uma necessidade de chamar uma interface HTTP e responder com um atraso de um segundo. Abaixo estão as formas de programação assíncrona e com corrotinas.

**Programação assíncrona com retorno de chamada**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // Chamar a interface HTTP
    $http->get('http://example.com/', function ($response) use ($connection) {
        // Enviar com atraso de um segundo
        Timer::add(1, function() use ($connection, $response) {
            // Enviar dados para o navegador
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**Programação com corrotinas**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // Chamar a interface HTTP
    $response = $http->get('http://example.com/');
    // Atraso de 1 segundo
    Timer::sleep(1);
    // Enviar dados
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **Observação**
> O código acima requer a instalação de `composer require workerman/http-client ^2.0.0`.

Ambas as abordagens acima executam de forma assíncrona e não bloqueante, com eficiência de execução. No entanto, a programação com corrotinas é mais fácil de se ler e manter do que o retorno de chamada assíncrono.

### Considerações sobre Fibra
* As corrotinas de Fibra não suportam a corotinização de funções de Pdo, Redis e funções de bloqueio internas do PHP. Isso significa que o uso dessas extensões e funções ainda resultará em chamadas bloqueantes.
* Os clientes de corrotinas de Fibra disponíveis atualmente incluem [workerman/http-client](../components/workerman-http-client.md) e [workerman/redis](../components/workerman-redis.md).

# Swoole Coroutines
O workerman v5 também suporta o uso do Swoole como driver de eventos subjacente.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// É necessário definir manualmente o Swoole como driver de eventos subjacente
Worker::$eventLoopClass = Workerman\Events\Swoole::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```

**Dicas**
* É recomendável usar o Swoole 5.0 ou versões posteriores
* Ao definir o Swoole como driver de eventos subjacente, o trabalho do workerman suporta as corrotinas do swoole
* Ao usar o Swoole como driver de eventos subjacente, não é necessário instalar a extensão de evento
* Por padrão, o Swoole não ativa as corrotinas com um clique, o que significa que as chamadas baseadas em Pdo, Redis, leitura/gravação de arquivos internos do PHP são bloqueantes
* Para ativar as corrotinas com um clique, é necessário chamar manualmente `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`

Para mais informações, consulte o [Manual do Swoole](https://wiki.swoole.com/).

Para mais informações, consulte o [Driver de Eventos](appendices/event.md).

# Sobre Corrotinas
Em primeiro lugar, não é necessário ter uma fé cega nas corrotinas. Quando o banco de dados, o Redis e outros sistemas de armazenamento estão na rede interna, muitas vezes as chamadas de vários processos em modo bloqueio podem ser mais rápidas do que as corrotinas. Com base nos dados de benchmarking dos últimos 3 anos do [techempower.com](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db), as chamadas de banco de dados em modo bloqueio no workerman têm um desempenho superior ao pool de conexões e corrotinas do Swoole, até mesmo ultrapassando em quase 1 vez o desempenho de estruturas de corrotinas como gin e echo em Go.

O workerman já aumentou o desempenho das aplicações em PHP várias vezes, e na maioria dos projetos do workerman, a adição de corrotinas pode não trazer um grande aumento de desempenho. Se o sistema tiver chamadas lentas, como chamadas de HTTP externas, considerar a utilização de corrotinas para aumentar o desempenho seria uma opção a considerar.
