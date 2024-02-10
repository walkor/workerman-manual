# Explication
À partir de la version 4.x, Workerman a renforcé son support des services HTTP. Il a introduit une classe de requête, une classe de réponse, une classe de session ainsi que [SSE](SSE.md). Si vous souhaitez utiliser les services HTTP de Workerman, il est fortement recommandé d'utiliser la version 4.x ou ultérieure.

**Notez que les utilisations ci-dessous sont pour la version 4.x de Workerman et ne sont pas compatibles avec la version 3.x.**

## Obtention de l'objet de requête
L'objet de requête est toujours obtenu dans la fonction de rappel onMessage, le cadre transmettra automatiquement l'objet Request en tant que second paramètre de la fonction de rappel.

**Exemple**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request est l'objet de requête, ici il n'effectue aucune opération sur l'objet de requête et renvoie directement "hello" au navigateur
    $connection->send("hello");
};

// Exécuter le worker
Worker::runAll();
```

Lorsque le navigateur accède à `http://127.0.0.1:8080`, il renverra "hello".

## Obtenir les paramètres de requête GET

**Obtenir tout le tableau GET**
```php
$get = $request->get();
```
Si la demande ne contient pas de paramètres GET, cela renverra un tableau vide.

**Obtenir une valeur spécifique du tableau GET**
```php
$name = $request->get('name');
```
Si le tableau GET ne contient pas cette valeur, cela renverra null.

Vous pouvez également passer une valeur par défaut en deuxième paramètre à la méthode get. Si la valeur correspondante n'est pas trouvée dans le tableau GET, elle renverra la valeur par défaut. Par exemple :
```php
$name = $request->get('name', 'tom');
```

**Exemple**
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

// Exécuter le worker
Worker::runAll();
```
## Obtenir la méthode de la requête
```php
$method = $request->method();
```
La valeur renvoyée peut être l'une des suivantes : `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD`.

**Exemple**
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

// Exécution du worker
Worker::runAll();
```

## Obtenir l'URI de la requête
```php
$uri = $request->uri();
```
Renvoie l'URI de la requête, comprenant la partie path et queryString.

**Exemple**
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

// Exécution du worker
Worker::runAll();
```
Lorsque le navigateur accède à `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, il renverra `/user/get.php?uid=10&type=2`.

## Obtenir le chemin de la requête
```php
$path = $request->path();
```
Renvoie la partie path de la requête.

**Exemple**
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

// Exécution du worker
Worker::runAll();
```
Lorsque le navigateur accède à `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, il renverra `/user/get.php`.

## Obtenir la queryString de la requête
```php
$query_string = $request->queryString();
```
Renvoie la partie queryString de la requête.

**Exemple**
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

// Exécution du worker
Worker::runAll();
```
Lorsque le navigateur accède à `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, il renverra `uid=10&type=2`.

## Obtenir la version HTTP de la requête
```php
$version = $request->protocolVersion();
```
Renvoie la chaîne `1.1` ou `1.0`.

**Exemple**
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

// Exécution du worker
Worker::runAll();
```

## Obtenir l'identifiant de session de la requête
```php
$sid = $request->sessionId();
```
Renvoie une chaîne composée de lettres et de chiffres.

**Exemple**
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

// Exécution du worker
Worker::runAll();
```
