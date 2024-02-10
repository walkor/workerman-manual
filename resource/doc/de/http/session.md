# Anleitung

Ab der Version 4.x hat Workerman die Unterstützung für HTTP-Services verbessert. Es wurden Request-Klassen, Response-Klassen, Session-Klassen und [SSE](SSE.md) eingeführt. Wenn Sie Workermans HTTP-Services nutzen möchten, wird dringend empfohlen, Workerman 4.x oder eine spätere Version zu verwenden.

**Bitte beachten Sie, dass die folgenden Anweisungen alle für die Verwendung in Workerman 4.x geschrieben wurden und nicht mit Workerman 3.x kompatibel sind.**

# Sitzungsobjekt abrufen
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'Tome');
    $connection->send($session->get('name'));
};

// Worker ausführen
Worker::runAll();
```
**Hinweise**
- Die Session muss vor dem Aufruf von `$connection->send()` bearbeitet werden.
- Die Session wird automatisch gespeichert, wenn das Objekt zerstört wird. Daher sollten Sie das Objekt, das von `$request->session()` zurückgegeben wird, nicht in globalen Arrays oder Klassenattributen speichern, da die Session dann nicht gespeichert wird.
- Die Session wird standardmäßig in einer Datei auf der Festplatte gespeichert. Zur Verbesserung der Leistung wird die Verwendung von Redis empfohlen.

## Alle Session-Daten abrufen
```php
$session = $request->session();
$all = $session->all();
```
Es wird ein Array zurückgegeben. Wenn keine Session-Daten vorhanden sind, wird ein leeres Array zurückgegeben.

**Beispiel**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'Tom');
    $connection->send(var_export($session->all(), true));
};

// Worker ausführen
Worker::runAll();
```

## Einen Wert aus der Session abrufen
```php
$session = $request->session();
$name = $session->get('name');
```
Wenn die Daten nicht vorhanden sind, wird null zurückgegeben.

Sie können auch einen Standardwert als zweiten Parameter an die `get`-Methode übergeben. Wenn im Session-Array kein entsprechender Wert gefunden wird, wird der Standardwert zurückgegeben. Beispiel:
```php
$session = $request->session();
$name = $session->get('name', 'Tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'Tom'));
};

// Worker ausführen
Worker::runAll();
```

## Session speichern
Verwenden Sie die `set`-Methode, um bestimmte Daten zu speichern.
```php
$session = $request->session();
$session->set('name', 'Tom');
```
Die `set`-Methode gibt keinen Wert zurück, da die Session automatisch gespeichert wird, wenn das Objekt zerstört wird.

Verwenden Sie die `put`-Methode, um mehrere Werte zu speichern.
```php
$session = $request->session();
$session->put(['name' => 'Tom', 'age' => 12]);
```
Auch die `put`-Methode gibt keinen Wert zurück.

**Beispiel**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'Tom');
    $connection->send($session->get('name'));
};

// Worker ausführen
Worker::runAll();
```

## Session-Daten löschen
Verwenden Sie die `forget`-Methode, um einzelne oder mehrere Session-Daten zu löschen.
```php
$session = $request->session();
// Ein Element löschen
$session->forget('name');
// Mehrere Elemente löschen
$session->forget(['name', 'age']);
```

Außerdem gibt es die Methode `delete`, die sich von `forget` unterscheidet, da `delete` nur ein Element löschen kann.
```php
$session = $request->session();
// Äquivalent zu $session->forget('name');
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// Worker ausführen
Worker::runAll();
```

## Einen Session-Wert abrufen und löschen
```php
$session = $request->session();
$name = $session->pull('name');
```
Dies hat den gleichen Effekt wie der folgende Code:
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
Wenn die entsprechende Session nicht vorhanden ist, wird null zurückgegeben.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```

## Alle Session-Daten löschen
```php
$request->session()->flush();
```
Es gibt keinen Rückgabewert, da die Session automatisch aus dem Speicher gelöscht wird, wenn das Objekt zerstört wird.

**Beispiel**
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

// Worker ausführen
Worker::runAll();
```

## Überprüfen, ob bestimmte Session-Daten vorhanden sind
```php
$session = $request->session();
$has = $session->has('name');
```
Wenn die entsprechende Session nicht vorhanden ist oder der Wert der Session null ist, wird false zurückgegeben, andernfalls true.

```php
$session = $request->session();
$has = $session->exists('name');
```
Der obige Code dient ebenfalls dazu, zu überprüfen, ob Session-Daten vorhanden sind. Der Unterschied besteht darin, dass `exists` auch dann true zurückgibt, wenn der Wert des Session-Elements null ist.
