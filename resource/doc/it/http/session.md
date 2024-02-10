# Descrizione

A partire dalla versione 4.x, workerman ha rafforzato il supporto ai servizi HTTP. Ha introdotto una classe di richiesta, una classe di risposta, una classe di sessione e [SSE](SSE.md). Se desideri utilizzare il servizio HTTP di workerman, ti consigliamo vivamente di utilizzare la versione 4.x o versioni successive.

**Si noti che quanto segue si riferisce all'uso della versione 4.x di workerman e non è compatibile con la versione 3.x.**

# Ottenere l'oggetto sessione
```php
$session = $request->session();
```

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Esegui il worker
Worker::runAll();
```
**Note**
- La sessione deve essere gestita prima della chiamata `$connection->send()`.
- La sessione verrà automaticamente salvata quando l'oggetto viene distrutto, quindi non salvare l'oggetto restituito da `$request->session()` in un array globale o nell'attributo di una classe, altrimenti la sessione potrebbe non essere salvata correttamente.
- Per impostazioni predefinite, la sessione viene archiviata su file su disco, ma per prestazioni migliori si consiglia di utilizzare Redis.


## Ottenere tutti i dati della sessione
```php
$session = $request->session();
$all = $session->all();
```
Restituisce un array. Se non ci sono dati di sessione, restituisce un array vuoto.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send(var_export($session->all(), true));
};

// Esegui il worker
Worker::runAll();
```


## Ottenere un valore specifico dalla sessione
```php
$session = $request->session();
$name = $session->get('name');
```
Restituisce null se i dati non esistono.

È anche possibile passare un valore predefinito al metodo get. Se l'array di sessione non contiene il valore corrispondente, verrà restituito il valore predefinito. Ad esempio:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```
**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// Esegui il worker
Worker::runAll();
```


## Memorizzare la sessione
Usa il metodo set per memorizzare un dato specifico nella sessione.
```php
$session = $request->session();
$session->set('name', 'tom');
```
Il metodo set non restituisce alcun valore, e la sessione verrà salvata automaticamente quando l'oggetto sessione viene distrutto.

Per memorizzare più valori, utilizzare il metodo put.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Anche in questo caso, il metodo put non restituisce alcun valore.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send($session->get('name'));
};

// Esegui il worker
Worker::runAll();
```


## Eliminare i dati della sessione
Per eliminare uno o più dati della sessione, utilizzare il metodo `forget`.
```php
$session = $request->session();
// Elimina un dato
$session->forget('name');
// Elimina più dati
$session->forget(['name', 'age']);
```

Inoltre, esiste il metodo `delete` che, a differenza del metodo `forget`, può eliminare solo un elemento.
```php
$session = $request->session();
// Equivale a $session->forget('name');
$session->delete('name');
```

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $request->session()->forget('name');
    $connection->send('ok');
};

// Esegui il worker
Worker::runAll();
```


## Ottenere e cancellare un valore specifico della sessione
```php
$session = $request->session();
$name = $session->pull('name');
```
Il risultato è lo stesso di questo codice
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
Se la sessione corrispondente non esiste, verrà restituito null.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->session()->pull('name'));
};

// Esegui il worker
Worker::runAll();
```


## Eliminare tutti i dati della sessione
```php
$request->session()->flush();
```
Non restituisce alcun valore, e la sessione verrà automaticamente rimossa dallo storage quando l'oggetto sessione viene distrutto.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $request->session()->flush();
    $connection->send('ok');
};

// Esegui il worker
Worker::runAll();
```


## Verificare se un dato della sessione esiste
```php
$session = $request->session();
$has = $session->has('name');
```
Se la sessione corrispondente non esiste o se il valore è null, restituirà false, altrimenti restituirà true.

```php
$session = $request->session();
$has = $session->exists('name');
```
Anche questo codice viene utilizzato per verificare se esiste un dato della sessione, l'unica differenza è che restituirà true anche se il valore dell'elemento della sessione è null.
