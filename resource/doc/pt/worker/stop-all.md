# stopAll
```php
void Worker::stopAll(void)
```

Para a parada do processo atual e saída.

> **Observação**
> `Worker::stopAll()` é usado para parar o processo atual. Após a saída do processo atual, o processo principal irá imediatamente iniciar um novo processo. Se você deseja parar todo o serviço do Workerman, por favor, chame `posix_kill(posix_getppid(), SIGINT)`.

### Parâmetros
Sem parâmetros

### Valor Retornado
Sem retorno

## Exemplo max_request

No exemplo abaixo, o sub-processo executa `stopAll` e sai após processar 1000 requisições, para reiniciar um novo processo completamente. Similar à propriedade max_request do php-fpm, é principalmente usado para lidar com vazamento de memória causado por bugs no código de negócios em PHP.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Máximo de 1000 requisições por processo
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Número de requisições já processadas
    static $request_count = 0;

    $connection->send('hello http');
    // Se o número de requisições atingir 1000
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * Encerra o processo atual, e o processo principal imediatamente inicia um novo processo completamente
         * para completar a reinicialização do processo
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
