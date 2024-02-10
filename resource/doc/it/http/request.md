# Introduzione
Da Workerman 4.x in poi è stata potenziata la compatibilità con i servizi HTTP, introducendo la classe di richiesta, la classe di risposta, la classe di sessione e [SSE](SSE.md). Se vuoi utilizzare il servizio HTTP di Workerman, si consiglia vivamente di utilizzare la versione 4.x di Workerman o una versione successiva.

**Si noti che le seguenti istruzioni si applicano solo alla versione 4.x di Workerman e non sono compatibili con la versione 3.x.**

## Ottenere l'oggetto richiesta
L'oggetto richiesta viene sempre ottenuto nella funzione di callback onMessage e il framework passerà automaticamente l'oggetto Request come secondo parametro alla funzione di callback.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request è l'oggetto di richiesta, qui non viene eseguita nessuna operazione sull'oggetto di richiesta, viene semplicemente restituito "hello" al browser
    $connection->send("hello");
};

// Avvia il worker
Worker::runAll();
```

Quando si accede a `http://127.0.0.1:8080` dal browser, verrà restituito "hello".

## Ottenere i parametri della richiesta GET

**Ottenere l'intero array GET**
```php
$get = $request->get();
```
Se la richiesta non contiene parametri GET, viene restituito un array vuoto.

**Ottenere un valore specifico dall'array GET**
```php
$name = $request->get('name');
```
Se l'array GET non contiene quel valore, viene restituito null.

È anche possibile passare un valore predefinito come secondo argomento al metodo get. Se l'array GET non contiene il valore corrispondente, verrà restituito il valore predefinito. Ad esempio:
```php
$name = $request->get('name', 'tom');
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
    $connection->send($request->get('name'));
};

// Avvia il worker
Worker::runAll();
```

Quando si accede a `http://127.0.0.1:8080?name=jerry&age=12` dal browser, verrà restituito "jerry".

## Ottenere i parametri della richiesta POST
**Ottenere l'intero array POST**
```php
$post = $request->post();
```
Se la richiesta non contiene parametri POST, viene restituito un array vuoto.

**Ottenere un valore specifico dall'array POST**
```php
$name = $request->post('name');
```
Se l'array POST non contiene quel valore, viene restituito null.

Come il metodo get, è possibile passare un valore predefinito come secondo argomento al metodo post. Se l'array POST non contiene il valore corrispondente, verrà restituito il valore predefinito. Ad esempio:
```php
$name = $request->post('name', 'tom');
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
    $post = $request->post();
    $connection->send(var_export($post, true));
};

// Avvia il worker
Worker::runAll();
```
## Ottenere il corpo grezzo della richiesta POST
```php
$post = $request->rawBody();
```
Questa funzionalità è simile all'operazione `file_get_contents("php://input");` in `php-fpm`, ed è utile per ottenere il corpo grezzo della richiesta HTTP quando si ottengono dati di richiesta POST in un formato diverso da `application/x-www-form-urlencoded`.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = json_decode($request->rawBody());
    $connection->send('hello');
};

// Avvia il worker
Worker::runAll();
```

## Ottenere l'header
**Ottenere l'intero array delle intestazioni (header)**
```php
$headers = $request->header();
```
Se la richiesta non contiene intestazioni (header), viene restituito un array vuoto. Si noti che tutte le chiavi sono in minuscolo.

**Ottenere un valore specifico dall'array delle intestazioni (header)**
```php
$host = $request->header('host');
```
Se l'array delle intestazioni (header) non contiene quel valore, viene restituito null. Si noti che tutte le chiavi sono in minuscolo.

Come per il metodo get, è possibile passare un valore predefinito come secondo argomento al metodo header. Se l'array delle intestazioni (header) non contiene il valore corrispondente, verrà restituito il valore predefinito. Ad esempio:
```php
$host = $request->header('host', 'localhost');
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
    if ($request->header('connection') === 'keep-alive') {
        $connection->send('hello');
    } else {
        $connection->close('hello');
    }    
};

// Avvia il worker
Worker::runAll();
```

## Ottenere i cookie
**Ottenere l'intero array dei cookie**
```php
$cookies = $request->cookie();
```
Se la richiesta non contiene cookie, viene restituito un array vuoto.

**Ottenere un valore specifico dall'array dei cookie**
```php
$name = $request->cookie('name');
```
Se l'array dei cookie non contiene quel valore, viene restituito null.

Come per il metodo get, è possibile passare un valore predefinito come secondo argomento al metodo cookie. Se l'array dei cookie non contiene il valore corrispondente, verrà restituito il valore predefinito. Ad esempio:
```php
$name = $request->cookie('name', 'tom');
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
    $cookie = $request->cookie();
    $connection->send(var_export($cookie, true));
};

// Avvia il worker
Worker::runAll();
```

## Ottenere i file caricati
**Ottenere l'intero array dei file caricati**
```php
$files = $request->file();
```
Il formato del file restituito è simile a:
```php
array (
    'avatar' => array (
            'name' => '123.jpg',
            'tmp_name' => '/tmp/workerman.upload.9hjR4w',
            'size' => 1196127,
            'error' => 0,
            'type' => 'application/octet-stream',
      ),
     'anotherfile' =>  array (
            'name' => '456.txt',
            'tmp_name' => '/tmp/workerman.upload.9sirSws',
            'size' => 490,
            'error' => 0,
            'type' => 'text/plain',
      )
)
```
Dove:
 
 - name è il nome del file
 - tmp_name è la posizione temporanea del file su disco
 - size è la dimensione del file
 - error è il [codice di errore](https://www.php.net/manual/zh/features.file-upload.errors.php)
 - type è il tipo MIME del file.

**Nota:**

 - le dimensioni dei file caricati sono limitate dalla [defaultMaxPackageSize](../tcp-connection/default-max-package-size.md), di default 10M, ma può essere modificato.
 - Dopo la fine della richiesta, i file verranno cancellati automaticamente.
 - Se non ci sono file caricati nella richiesta, viene restituito un array vuoto.

### Ottenere un file specifico caricato
```php
$avatar_file = $request->file('avatar');
```
Restituisce qualcosa del genere
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
Se il file caricato non esiste, viene restituito null.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = $request->file('avatar');
    if ($file && $file['error'] === UPLOAD_ERR_OK) {
        rename($file['tmp_name'], '/home/www/web/public/123.jpg');
        $connection->send('ok');
        return;
    }
    $connection->send('upload fail');
};

// Avvia il worker
Worker::runAll();
```

## Ottenere l'host
Ottieni le informazioni di host della richiesta.
```php
$host = $request->host();
```
Se l'indirizzo della richiesta è su una porta non standard (diversa da 80 o 443), le informazioni di host potrebbero includere la porta, ad esempio `example.com:8080`. Se non è necessaria la porta, è possibile passare `true` come primo argomento.

```php
$host = $request->host(true);
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
    $connection->send($request->host());
};

// Avvia il worker
Worker::runAll();
```
Quando si accede a `http://127.0.0.1:8080?name=tom` dal browser, verrà restituito `127.0.0.1:8080`.
## Ottenere il metodo della richiesta
```php
$method = $request->method();
```
Il valore restituito potrebbe essere uno tra `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD`.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->method());
};

// Avvia il worker
Worker::runAll();
```
## Ottenere l'URI della richiesta
```php
$uri = $request->uri();
```
Restituisce l'URI della richiesta, inclusi il percorso (path) e la stringa di query.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->uri());
};

// Avvia il worker
Worker::runAll();
```
Quando si accede tramite browser a `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, verrà restituito `/user/get.php?uid=10&type=2`.

## Ottenere il percorso della richiesta
```php
$path = $request->path();
```
Restituisce la parte del percorso della richiesta.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->path());
};

// Avvia il worker
Worker::runAll();
```
Quando si accede tramite browser a `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, verrà restituito `/user/get.php`.

## Ottenere la stringa di query della richiesta
```php
$query_string = $request->queryString();
```
Restituisce la parte della stringa di query della richiesta.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->queryString());
};

// Avvia il worker
Worker::runAll();
```
Quando si accede tramite browser a `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, verrà restituito `uid=10&type=2`.

## Ottenere la versione HTTP della richiesta
```php
$version = $request->protocolVersion();
```
Restituisce la stringa `1.1` o `1.0`.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->protocolVersion());
};

// Avvia il worker
Worker::runAll();
```

## Ottenere l'ID della sessione della richiesta
```php
$sid = $request->sessionId();
```
Restituisce una stringa composta da lettere e numeri.

**Esempio**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->sessionId());
};

// Avvia il worker
Worker::runAll();
```
