# stopAll
```php
void Worker::stopAll(void)
```

Arrête le processus actuel et quitte.

> **Remarque**
> `Worker::stopAll()` est utilisé pour arrêter le processus actuel. Une fois que le processus actuel s'arrête, le processus principal lancera immédiatement un nouveau processus. Si vous souhaitez arrêter l'ensemble du service workerman, veuillez appeler `posix_kill(posix_getppid(), SIGINT)`.

### Paramètres
Aucun paramètre

### Valeur de retour
Aucune valeur de retour

## Exemple max_request

Dans l'exemple suivant, le sous-processus exécute stopAll après avoir traité 1000 requêtes, afin de redémarrer un tout nouveau processus. Similaire à la propriété max_request de php-fpm, principalement utilisée pour résoudre les problèmes de fuites de mémoire causés par des bugs dans le code métier PHP.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Maximum de 1000 requêtes par processus
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Nombre de requêtes traitées
    static $request_count = 0;

    $connection->send('hello http');
    // Si le nombre de requêtes atteint 1000
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * Arrête le processus actuel, le processus principal lancera immédiatement un nouveau processus tout neuf pour assurer la reprise du processus
         * et ainsi réaliser le redémarrage du processus
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
