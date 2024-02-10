# Identifiant

## Description :
```php
int Connection::$id
```

L'identifiant de la connexion. Il s'agit d'un entier en auto-incrémentation.

Remarque : Workerman est multiprocessus, chaque processus maintient un identifiant de connexion auto-incrémenté, de sorte que les identifiants de connexion entre plusieurs processus peuvent se répéter. Si vous souhaitez des identifiants de connexion non répétés, vous pouvez réaffecter la valeur de `connection->id` en fonction de vos besoins, par exemple en ajoutant un préfixe `worker->id`.

## Voir aussi
[La propriété `connections` de Worker](../worker/connections.md)

## Exemple
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// Exécute le worker
Worker::runAll();
```
