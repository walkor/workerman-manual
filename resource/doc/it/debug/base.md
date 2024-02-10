# Debug di base

WorkerMan ha due modalità di esecuzione, la modalità di debug e la modalità daemon.

Esegui ```php start.php start``` per entrare nella modalità di debug. In questo modo, le funzioni di stampa come ```echo, var_dump, var_export``` nel codice verranno visualizzate nel terminale. Si noti che se si esegue ```php start.php start```, tutti i processi di WorkerMan verranno chiusi quando si chiude il terminale.

Mentre l'esecuzione di ```php start.php start -d``` entra nella modalità daemon, cioè la modalità di esecuzione effettiva sul sito, e non è influenzata dalla chiusura del terminale.


Se si desidera vedere le funzioni di stampa come ```echo, var_dump, var_export``` anche quando si esegue in modalità daemon, è possibile impostare la proprietà Worker::$stdoutFile, ad esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Inoltra l'output dello schermo al file specificato in Worker::$stdoutFile
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

In questo modo, tutti i risultati delle funzioni di stampa come ```echo, var_dump, var_export``` verranno scritti nel file specificato da ```Worker::$stdoutFile```. Si noti che il percorso specificato da ```Worker::$stdoutFile``` deve avere i permessi di scrittura.
