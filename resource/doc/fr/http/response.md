# Explication

Depuis la version 4.x, Workerman a renforcé son support pour les services HTTP. Il a introduit une classe de requête, une classe de réponse, une classe de session et le [SSE](SSE.md). Si vous souhaitez utiliser les services HTTP de Workerman, il est fortement recommandé d'utiliser la version 4.x ou une version ultérieure.

**Veuillez noter que les exemples suivants sont basés sur l'utilisation de Workerman version 4.x, et ne sont pas compatibles avec la version 3.x.**

## Remarque

- Sauf pour l'envoi de réponses en morceaux (chunk) ou SSE, il n'est pas autorisé d'envoyer plusieurs réponses dans une seule requête, c'est-à-dire qu'il n'est pas autorisé d'appeler plusieurs fois `$connection->send()` dans une seule requête.
- Chaque requête doit finalement appeler `$connection->send()` pour envoyer une réponse, sinon le client attendra indéfiniment.

## Réponse rapide
Lorsque vous n'avez pas besoin de modifier le code d'état HTTP (200 par défaut), ou de personnaliser les en-têtes ou les cookies, vous pouvez envoyer directement une chaîne de caractères au client pour répondre.

**Exemple**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Envoyer directement "this is body" au client
    $connection->send("this is body");
};

// Lancer le worker
Worker::runAll();
```

## Modifier le code d'état
Lorsque vous devez personnaliser le code d'état, les en-têtes ou les cookies, vous devez utiliser la classe de réponse `Workerman\Protocols\Http\Response`. Par exemple, dans l'exemple ci-dessous, lors de l'accès à l'URL `/404`, le code d'état 404 est renvoyé avec le corps `<h1>Désolé, le fichier n'existe pas</h1>`.

**Exemple**
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
        $connection->send(new Response(404, [], '<h1>Désolé, le fichier n\'existe pas</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// Lancer le worker
Worker::runAll();
```
Une fois que la classe `Response` est initialisée, pour modifier le code d'état, utilisez la méthode suivante.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## Envoyer des en-têtes
De même, pour envoyer des en-têtes, vous devez utiliser la classe de réponse `Workerman\Protocols\Http\Response`.

**Exemple**
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

// Lancer le worker
Worker::runAll();
```
Une fois que la classe `Response` est initialisée, pour ajouter ou modifier un en-tête, utilisez la méthode suivante.
```php
$response = new Response(200);
// Ajouter ou modifier un en-tête
$response->header('Content-Type', 'text/html');
// Ajouter ou modifier plusieurs en-têtes
$response->withHeaders([
    'Content-Type' => 'application/json',
    'X-Header-One' => 'Header Value'
]);
$connection->send($response);
```

## Redirection
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
