# Erklärung

Ab der Version 4.x hat Workerman die Unterstützung für HTTP-Dienste verstärkt. Es wurden eine Request-Klasse, eine Response-Klasse, eine Session-Klasse und [SSE](SSE.md) eingeführt. Wenn Sie die HTTP-Dienste von Workerman nutzen möchten, wird dringend empfohlen, die Verwendung von Workerman 4.x oder höheren Versionen in Betracht zu ziehen.

**Bitte beachten Sie, dass die folgenden Anleitungen alle für die Verwendung mit Workerman 4.x geschrieben sind und nicht mit Workerman 3.x kompatibel sind.**

# Hinweis

- Es ist nicht erlaubt, in einer Anfrage mehrmals eine Antwort zu senden, es sei denn, es handelt sich um eine Chunk- oder SSE-Antwort. Das bedeutet, dass mehrere Aufrufe von `$connection->send()` in einer Anfrage nicht erlaubt sind.
- Für jede Anfrage muss am Ende einmal `$connection->send()` aufgerufen werden, um eine Antwort zu senden, ansonsten wartet der Client endlos.

## Schnelle Antwort

Wenn Sie den HTTP-Statuscode nicht ändern müssen (Standardwert: 200) oder Kopfzeilen und Cookies anpassen möchten, können Sie einfach eine Zeichenfolge an den Client senden, um die Antwort abzuschließen.

**Beispiel**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Direktes Senden von "this is body" an den Client
    $connection->send("this is body");
};

// Worker ausführen
Worker::runAll();
```

## Statuscode ändern

Wenn Sie den Statuscode anpassen oder benutzerdefinierte Header und Cookies benötigen, müssen Sie die Response-Klasse `Workerman\Protocols\Http\Response` verwenden. Das folgende Beispiel zeigt, wie bei einem Zugriff auf den Pfad `/404` der Statuscode 404 und der Body `<h1>Entschuldigung, die Datei existiert nicht</h1>` zurückgegeben werden.

**Beispiel**
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
        $connection->send(new Response(404, [], '<h1>Entschuldigung, die Datei existiert nicht</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// Worker ausführen
Worker::runAll();
```
Um den Statuscode nach der Initialisierung der `Response`-Klasse zu ändern, verwenden Sie die folgende Methode.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## Header senden

Ebenso muss für das Senden von Headern die Response-Klasse `Workerman\Protocols\Http\Response` verwendet werden.

**Beispiel**
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
        'X-Header-One' => 'Header Value'
    ], 'this is body');
    $connection->send($response);
};

// Worker ausführen
Worker::runAll();
```
Nach der Initialisierung der `Response`-Klasse können Sie Header hinzufügen oder ändern, indem Sie die folgende Methode verwenden.
```php
$response = new Response(200);
// Einzelnen Header hinzufügen oder ändern
$response->header('Content-Type', 'text/html');
// Mehrere Header hinzufügen oder ändern
$response->withHeaders([
    'Content-Type' => 'application/json',
    'X-Header-One' => 'Header Value'
]);
$connection->send($response);
```

## Umleitung

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)

$worker = new Worker('http://0.0.0.0:8080');
$worker->onMessage = function($connection, $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};
Worker::runAll();
```

## Cookie senden

Ebenso muss für das Senden von Cookies die Response-Klasse `Workerman\Protocols\Http\Response` verwendet werden.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```

## Datei senden

Ebenso muss für das Senden von Dateien die Response-Klasse `Workerman\Protocols\Http\Response` verwendet werden.

Die folgende Methode wird zum Senden von Dateien verwendet
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- Workerman unterstützt das Senden von sehr großen Dateien.
- Für große Dateien (über 2 MB) liest Workerman die gesamte Datei nicht auf einmal in den Speicher, sondern liest und sendet die Datei zu geeigneten Zeitpunkten in Abschnitten.
- Workerman optimiert die Dateilese- und -sendegeschwindigkeit basierend auf der Empfangsgeschwindigkeit des Clients, um eine schnelle Dateiübertragung zu gewährleisten und den Speicherbedarf zu minimieren.
- Die Datenübertragung erfolgt nicht blockierend und beeinträchtigt die Verarbeitung anderer Anfragen nicht.
- Beim Senden von Dateien wird automatisch der Header `Last-Modified` hinzugefügt, damit der Server beim nächsten Request prüfen kann, ob eine 304-Antwort gesendet werden kann, um die Dateiübertragung zu optimieren und die Leistung zu verbessern.
- Die gesendete Datei wird automatisch mit dem passenden `Content-Type`-Header an den Browser gesendet.
- Wenn die Datei nicht existiert, wird automatisch eine 404-Antwort generiert

**Beispiel**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = '/your/path/of/file';
    // Überprüfen des if-modified-since-Headers, um festzustellen, ob die Datei geändert wurde
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // Wenn die Datei nicht geändert wurde, wird eine 304-Antwort gesendet
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // Wenn die Datei geändert wurde oder kein if-modified-since-Header vorhanden ist, wird die Datei gesendet
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// Worker ausführen
Worker::runAll();
```

## Senden von HTTP-Chunk-Daten
- Zuerst muss eine Response-Antwort mit dem Header `Transfer-Encoding: chunked` an den Client gesendet werden.
- Anschließend können weitere Chunk-Daten mit der Klasse `Workerman\Protocols\Http\Chunk` gesendet werden.
- Am Ende muss ein leerer Chunk gesendet werden, um die Antwort zu beenden.

**Beispiel**
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
    // Senden einer Response-Antwort mit dem Header `Transfer-Encoding: chunked`
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // Weitere Chunk-Daten werden mit der Klasse `Workerman\Protocols\Http\Chunk` gesendet
    $connection->send(new Chunk('Erster Datenteil'));
    $connection->send(new Chunk('Zweiter Datenteil'));
    $connection->send(new Chunk('Dritter Datenteil'));
   //  Am Ende muss ein leerer Chunk gesendet werden, um die Antwort zu beenden
    $connection->send(new Chunk(''));
};

// Worker ausführen
Worker::runAll();
```
