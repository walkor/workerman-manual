# runAll
```php
void Worker::runAll(void)
```
Exécute toutes les instances Worker.

**Remarque :** Une fois que Worker::runAll() est exécuté, il sera bloqué de manière permanente, ce qui signifie que le code après Worker::runAll() ne sera pas exécuté. Toutes les instances de Worker doivent être initialisées avant Worker::runAll().

### Paramètres
Aucun paramètre

### Valeur de retour
Aucune valeur de retour

## Exemple : Exécuter plusieurs instances de Worker

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// Exécute toutes les instances de Worker
Worker::runAll();
```

**Remarque :** La version Windows de workerman ne prend pas en charge l'instanciation de plusieurs Workers dans le même fichier. L'exemple ci-dessus ne peut pas être exécuté dans la version Windows de workerman.

La version Windows de workerman nécessite que les initialisations des instances de Worker soient placées dans des fichiers différents, comme ci-dessous :

start_http.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

// Exécute toutes les instances de Worker (ici, il n'y a qu'une seule instance)
Worker::runAll();
```

start_websocket.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// Exécute toutes les instances de Worker (ici, il n'y a qu'une seule instance)
Worker::runAll();
```
