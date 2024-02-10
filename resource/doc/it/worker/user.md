```php
# utente

## Descrizione:
string Worker::$user

Imposta con quale utente deve essere eseguita l'istanza corrente del Worker. Questa proprietà è valida solo quando l'utente corrente è root. Se non viene impostata, verrà eseguita con l'utente corrente. 

Si consiglia di impostare `$user` con un utente con pochi privilegi, ad esempio www-data, apache, nobody, ecc.

Nota: questa proprietà deve essere impostata prima di eseguire `Worker::runAll();`. Questa funzionalità non è supportata su sistemi Windows.


## Esempio

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Imposta l'utente di esecuzione dell'istanza
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Avvio del Worker...\n";
};
// Esegui il worker
Worker::runAll();
```
