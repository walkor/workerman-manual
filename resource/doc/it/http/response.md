# Documentazione

A partire dalla versione 4.x, Workerman ha potenziato il supporto ai servizi HTTP. È stato introdotto una classe di richiesta, una classe di risposta, una classe di sessione e anche (SSE)[SSE.md]. Se si desidera utilizzare il servizio HTTP di Workerman, si consiglia vivamente di utilizzare la versione 4.x di Workerman o versioni successive.

**Si noti che di seguito si fa riferimento all'uso della versione 4.x di Workerman e non è compatibile con la versione 3.x.**

# Note

- A meno che non si stiano inviando risposte chunk o SSE, non è consentito inviare più di una volta una risposta in una singola richiesta, ovvero non è consentito chiamare `$connection->send()` più volte in una singola richiesta.
- Ogni richiesta deve alla fine chiamare una volta `$connection->send()` per inviare la risposta, altrimenti il client rimarrà in attesa.

## Risposta rapida
Quando non è necessario modificare il codice di stato HTTP (impostato di default a 200), o personalizzare l'header o i cookie, è possibile inviare direttamente una stringa al client per completare la risposta.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Invia direttamente "this is body" al client
    $connection->send("this is body");
};

// Avvia il worker
Worker::runAll();
```

## Modifica del codice di stato
Quando è necessario personalizzare il codice di stato, l'header o i cookie, è necessario utilizzare la classe di risposta `Workerman\Protocols\Http\Response`. Ad esempio, l'esempio seguente restituisce il codice di stato 404 e il contenuto del corpo è `<h1>Scusa, il file non esiste</h1>` quando si accede al percorso `/404`.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->path() === '/404') {
        $connection->send(new Response(404, [], '<h1>Scusa, il file non esiste</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// Avvia il worker
Worker::runAll();
```
Una volta inizializzata la classe `Response`, è possibile modificare il codice di stato utilizzando il metodo seguente.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## Invio header
Anche per l'invio dell'header è necessario utilizzare la classe di risposta `Workerman\Protocols\Http\Response`.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [
        'Content-Type' => 'text/html',
        'X-Header-One' => 'Valore dell'header'
    ], 'this is body');
    $connection->send($response);
};

// Avvia il worker
Worker::runAll();
```
Una volta inizializzata la classe `Response`, è possibile aggiungere o modificare l'header utilizzando il metodo seguente.
```php
$response = new Response(200);
// Aggiunge o modifica un header
$response->header('Content-Type', 'text/html');
// Aggiunge o modifica più header
$response->withHeaders([
    'Content-Type' => 'application/ json',
    'X-Header-One' => 'Valore dell'header'
]);
$connection->send($response);
```

## Redirezione
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};

// Avvia il worker
Worker::runAll();
```

## Invio cookie
Anche per l'invio dei cookie è necessario utilizzare la classe di risposta `Workerman\Protocols\Http\Response`.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [], 'this is body');
    $response->cookie('name', 'tom');
    $connection->send($response);
};

// Avvia il worker
Worker::runAll();
```

## Invio di file
Anche per l'invio di file è necessario utilizzare la classe di risposta `Workerman\Protocols\Http\Response`.

Per inviare un file, è possibile utilizzare il seguente metodo
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- Workerman supporta l'invio di file di grandi dimensioni
- Per i file di grandi dimensioni (oltre 2 MB), Workerman non carica l'intero file in memoria una sola volta, ma legge e invia il file in segmenti in momenti opportuni
- Workerman ottimizza la velocità di lettura del file in base alla velocità con cui il client riceve i dati, garantendo l'invio più veloce possibile riducendo al minimo l'uso della memoria
- L'invio dei dati è non bloccante e non interferisce con la gestione di altre richieste
- Quando si invia un file, viene automaticamente aggiunto l'header `Last-Modified` per consentire al server di verificare se inviare una risposta 304 la prossima volta per risparmiare trasferimento dati e aumentare le prestazioni
- Il file inviato utilizza automaticamente l'header `Content-Type` appropriato per essere inviato al browser
- Se il file non esiste, viene automaticamente gestito come risposta 404

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = '/il/tuo/percorso/del/file';
    // Verifica l'header if-modified-since per determinare se il file è stato modificato
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // Se il file non è stato modificato restituisci una risposta 304
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // Se il file è stato modificato o non c'è l'header if-modified-since, invia il file
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// Avvia il worker
Worker::runAll();
```

## Invio di dati http chunk
 - È necessario inviare prima una risposta `Response` con l'header `Transfer-Encoding: chunked` al client
 - Per inviare dati chunk successivi, utilizzare la classe `Workerman\Protocols\Http\Chunk`
 - Infine, è necessario inviare un chunk vuoto per terminare la risposta

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Protocols\Http\Chunk;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Invia prima una risposta `Response` con l'header `Transfer-Encoding: chunked`
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // Utilizza la classe `Workerman\Protocols\Http\Chunk` per inviare dati chunk successivi
    $connection->send(new Chunk('Primo blocco di dati'));
    $connection->send(new Chunk('Secondo blocco di dati'));
    $connection->send(new Chunk('Terzo blocco di dati'));
   // Alla fine è necessario inviare un chunk vuoto per terminare la risposta
    $connection->send(new Chunk(''));
};

// Avvia il worker
Worker::runAll();
```
