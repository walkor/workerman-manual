# Erklärung
Ab der Version 4.x hat Workerman die Unterstützung für HTTP-Dienste erweitert. Es wurden Request-Klasse, Response-Klasse, Session-Klasse und [SSE](SSE.md) eingeführt. Wenn Sie Workerman für HTTP-Dienste verwenden möchten, wird dringend empfohlen, Workerman 4.x oder höhere Versionen zu verwenden.

**Beachten Sie, dass dies alle Verwendungsweisen der Workerman 4.x-Version sind und nicht mit Workerman 3.x kompatibel sind.**

## Abrufen des Anfrageobjekts
Das Anfrageobjekt wird immer im onMessage-Rückruffunktion erhalten, das Framework überträgt automatisch das Request-Objekt als zweiten Parameter in die Rückruffunktion.

**Beispiel**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request ist das Anfrageobjekt, hier wird kein direkter Vorgang auf das Anfrageobjekt ausgeführt, sondern direkt "hello" an den Browser zurückgesendet.
    $connection->send("hello");
};

// Worker ausführen
Worker::runAll();
```

Wenn der Browser `http://127.0.0.1:8080` aufruft, wird "hello" zurückgegeben.

## Abrufen der GET-Anfrageparameter

**Abrufen des gesamten GET-Arrays**
```php
$get = $request->get();
```
Wenn keine GET-Parameter in der Anfrage enthalten sind, wird ein leeres Array zurückgegeben.

**Abrufen eines Werts aus dem GET-Array**
```php
$name = $request->get('name');
```
Wenn der GET-Array diesen Wert nicht enthält, wird null zurückgegeben.

Sie können auch einen Standardwert als zweites Argument an die `get`-Methode übergeben, der zurückgegeben wird, wenn der entsprechende Wert im GET-Array nicht gefunden wird. Zum Beispiel:
```php
$name = $request->get('name', 'tom');
```

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```

Wenn der Browser `http://127.0.0.1:8080?name=jerry&age=12` aufruft, wird `jerry` zurückgegeben.

## Abrufen der POST-Anfrageparameter
**Abrufen des gesamten POST-Arrays**
```php
$post = $request->post();
```
Wenn keine POST-Parameter in der Anfrage enthalten sind, wird ein leeres Array zurückgegeben.

**Abrufen eines Werts aus dem POST-Array**
```php
$name = $request->post('name');
```
Wenn der POST-Array diesen Wert nicht enthält, wird null zurückgegeben.

Wie bei der `get`-Methode können Sie der `post`-Methode auch einen Standardwert als zweites Argument übergeben, der zurückgegeben wird, wenn der entsprechende Wert im POST-Array nicht gefunden wird. Zum Beispiel:
```php
$name = $request->post('name', 'tom');
```

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```

## Abrufen des ursprünglichen POST-Anfragepakets
```php
$post = $request->rawBody();
```
Diese Funktion ähnelt der Verwendung von `file_get_contents("php://input")` in `php-fpm`. Es dient zum Abrufen des ursprünglichen HTTP-Anfragekörpers. Dies ist nützlich, um POST-Anfragedaten in einem Format zu erhalten, das nicht `application/x-www-form-urlencoded` ist.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```
## Header abrufen
**Abrufen des gesamten Header-Arrays**
```php
$headers = $request->header();
```
Wenn die Anfrage keine Header-Parameter enthält, wird ein leeres Array zurückgegeben. Beachten Sie, dass alle Keys klein geschrieben sind.

**Abrufen eines Werts aus dem Header-Array**
```php
$host = $request->header('host');
```
Wenn der Header-Array diesen Wert nicht enthält, wird null zurückgegeben. Beachten Sie, dass alle Keys klein geschrieben sind.

Wie bei der `get`-Methode können Sie der `header`-Methode auch einen Standardwert als zweites Argument übergeben, der zurückgegeben wird, wenn der entsprechende Wert im Header-Array nicht gefunden wird. Zum Beispiel:
```php
$host = $request->header('host', 'localhost');
```

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```

## Cookie abrufen
**Abrufen des gesamten Cookie-Arrays**
```php
$cookies = $request->cookie();
```
Wenn die Anfrage keine Cookie-Parameter enthält, wird ein leeres Array zurückgegeben.

**Abrufen eines Werts aus dem Cookie-Array**
```php
$name = $request->cookie('name');
```
Wenn das Cookie-Array diesen Wert nicht enthält, wird null zurückgegeben.

Wie bei der `get`-Methode können Sie auch der `cookie`-Methode einen Standardwert als zweites Argument übergeben, der zurückgegeben wird, wenn der entsprechende Wert im Cookie-Array nicht gefunden wird. Zum Beispiel:
```php
$name = $request->cookie('name', 'tom');
```

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```

## Hochgeladene Dateien abrufen
**Abrufen des gesamten Array der hochgeladenen Dateien**
```php
$files = $request->file();
```
Das zurückgegebene Dateiformat sieht ähnlich aus:
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
Dabei:
 
 - name ist der Dateiname
 - tmp_name ist der temporäre Dateispeicherort auf der Festplatte
 - size ist die Dateigröße
 - error ist der [Fehlercode](https://www.php.net/manual/zh/features.file-upload.errors.php)
 - type ist der MIME-Typ der Datei.

**Hinweis:**

 - Die Größe der hochgeladenen Dateien unterliegt der Einschränkung von [defaultMaxPackageSize](../tcp-connection/default-max-package-size.md), standardmäßig 10 MB, kann jedoch geändert werden.

 - Nach Abschluss der Anfrage werden die Dateien automatisch gelöscht.

 - Wenn die Anfrage keine hochgeladenen Dateien enthält, wird ein leeres Array zurückgegeben.

### Abrufen einer bestimmten hochgeladenen Datei
```php
$avatar_file = $request->file('avatar');
```
Gibt Ähnliches zurück
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
Wenn die hochgeladene Datei nicht vorhanden ist, wird null zurückgegeben.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```
## Host abrufen
Abrufen der Host-Informationen der Anfrage.
```php
$host = $request->host();
```
Wenn die Adresse der Anfrage nicht der Standard-Port 80 oder 443 ist, kann die Host-Information einen Port enthalten, z.B. `example.com:8080`. Wenn kein Port benötigt wird, kann dem ersten Argument `true` übergeben werden.

```php
$host = $request->host(true);
```
**Beispiel**
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

// Worker ausführen
Worker::runAll();
```
Wenn der Browser `http://127.0.0.1:8080?name=tom` aufruft, wird `127.0.0.1:8080` zurückgegeben.
## Abrufen der Anfragemethode
```php
$method = $request->method();
```
Der Rückgabewert kann einer von `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD` sein.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```

## Abrufen der Anfrage-URI
```php
$uri = $request->uri();
```
Gibt die angeforderte URI zurück, einschließlich Pfad- und Query-String-Teile.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```
Wenn der Browser beispielsweise auf `http://127.0.0.1:8080/user/get.php?uid=10&type=2` zugreift, wird `/user/get.php?uid=10&type=2` zurückgegeben.

## Abrufen des Anfragepfads
```php
$path = $request->path();
```
Gibt den angeforderten Pfadteil zurück.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```
Wenn der Browser beispielsweise auf `http://127.0.0.1:8080/user/get.php?uid=10&type=2` zugreift, wird `/user/get.php` zurückgegeben.

## Abrufen der Anfrage-Query-String
```php
$query_string = $request->queryString();
```
Gibt den angeforderten Query-String-Teil zurück.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```
Wenn der Browser beispielsweise auf `http://127.0.0.1:8080/user/get.php?uid=10&type=2` zugreift, wird `uid=10&type=2` zurückgegeben.

## Abrufen der Anfrage-HTTP-Version
```php
$version = $request->protocolVersion();
```
Gibt den String `1.1` oder `1.0` zurück.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```

## Abrufen der Anfrage-Session-ID
```php
$sid = $request->sessionId();
```
Gibt einen String zurück, der aus Buchstaben und Zahlen besteht.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```
