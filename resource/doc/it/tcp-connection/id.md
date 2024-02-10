# id

## Descrizione
```php
int Connection::$id
```

L'ID della connessione. Si tratta di un intero incrementale.

Nota: workerman è multi-processo, quindi ogni processo manterrà un ID di connessione incrementale, il che significa che ci potrebbero essere duplicati di ID di connessione tra i processi. Se si desidera un ID di connessione non duplicato, è possibile assegnare nuovamente un valore a connection->id in base alle esigenze, ad esempio aggiungendo un prefisso worker->id.

## Vedi anche
[La proprietà connections di Worker](../worker/connections.md)

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// Avvia il worker
Worker::runAll();
```
