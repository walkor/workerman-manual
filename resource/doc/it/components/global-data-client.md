# Client del componente GlobalData
**``` (Richiede Workerman versione >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

Istanzia un oggetto client \GlobalData\Client. Condividi i dati tra i processi assegnando le proprietà sull'oggetto del client.

### Parametri
Indirizzo del server GlobalData, nel formato ```<indirizzo IP>:<porta>```, ad esempio ```127.0.0.1:2207```.

Se è un cluster di server GlobalData, passare un array di indirizzi, ad esempio ```array('10.0.0.10:2207', '10.0.0.0.11:2207')```.

## Spiegazione
Supporta operazioni di assegnazione, lettura, isset, unset.
Supporta anche operazioni atomiche cas.

## Esempio

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Server GlobalData
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// Quando il processo inizia
$worker->onWorkerStart = function()
{
    // Inizializza un client global data globale
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// Ogni volta che il server riceve un messaggio
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Cambia il valore di $global->somedata, altri processi condivideranno questa variabile $global->somedata
    global $global;
    echo "ora global->somedata=".var_export($global->somedata, true)."\n";
    echo "imposta \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### Uso completo (utilizzabile anche in ambiente php-fpm)
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));

```

## Attenzione:
Il componente GlobalData non può condividere dati di tipo di risorsa, come connessioni mysql, connessioni socket, ecc.

Se stai utilizzando il client GlobalData in un ambiente Workerman, istanzia l'oggetto GlobalData/Client nei callback onXXX, ad esempio in onWorkerStart.

Non è possibile operare sulle variabili condivise in questo modo.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
Si può fare in questo modo
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
