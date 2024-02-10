# Explication

À partir de la version 4.x, Workerman a renforcé le support des services HTTP. Il a introduit une classe de requête, une classe de réponse, une classe de session ainsi que [SSE](SSE.md). Si vous souhaitez utiliser les services HTTP de Workerman, il est fortement recommandé d'utiliser la version 4.x de Workerman ou une version ultérieure.

**Veuillez noter que toutes les utilisations suivantes sont pour la version 4.x de Workerman et ne sont pas compatibles avec la version 3.x.**

# Obtenir l'objet session
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Exécution du worker
Worker::runAll();
```
**Remarques**
- La session doit être manipulée avant l'appel de `$connection->send()`.
- La session est automatiquement sauvegardée lors de la destruction de l'objet, donc ne conservez pas l'objet retourné par `$request->session()` dans un tableau global ou un membre de classe, sinon la session ne sera pas sauvegardée.
- Par défaut, la session est stockée dans des fichiers sur disque. Pour de meilleures performances, l'utilisation de Redis est recommandée.

## Obtenir toutes les données de session
```php
$session = $request->session();
$all = $session->all();
```
Retourne un tableau. Si aucune donnée de session n'est présente, un tableau vide est renvoyé.

**Exemple**
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

// Exécution du worker
Worker::runAll();
```

## Obtenir une valeur spécifique de la session
```php
$session = $request->session();
$name = $session->get('name');
```
Retourne null si les données n'existent pas.

Vous pouvez également passer une valeur par défaut en deuxième paramètre à la méthode get. La valeur par défaut sera renvoyée si la clé n'existe pas dans le tableau de session. Par exemple :
```php
$session = $request->session();
$name = $session->get('name', 'tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// Exécution du worker
Worker::runAll();
```

## Stocker une session
Utilisez la méthode set pour stocker des données.
```php
$session = $request->session();
$session->set('name', 'tom');
```
La méthode set n'a pas de valeur de retour et la session est automatiquement sauvegardée lorsque l'objet session est détruit.

Pour stocker plusieurs valeurs, utilisez la méthode put.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
De même, la méthode put n'a pas de valeur de retour.

**Exemple**
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

// Exécution du worker
Worker::runAll();
```

## Supprimer des données de session
Utilisez la méthode `forget` pour supprimer une ou plusieurs données de session.
```php
$session = $request->session();
// Supprimer un élément
$session->forget('name');
// Supprimer plusieurs éléments
$session->forget(['name', 'age']);
```

En outre, le système propose également la méthode delete, qui diffère de la méthode forget en ce qu'elle peut uniquement supprimer un élément.
```php
$session = $request->session();
// Équivaut à $session->forget('name');
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// Exécution du worker
Worker::runAll();
```

## Obtenir et supprimer une valeur de session
```php
$session = $request->session();
$name = $session->pull('name');
```
Cela a le même effet que le code suivant :
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
Si la session correspondante n'existe pas, null est retourné.

**Exemple**
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

// Exécution du worker
Worker::runAll();
```

## Supprimer toutes les données de session
```php
$request->session()->flush();
```
Aucune valeur de retour, la session est automatiquement supprimée du stockage lors de la destruction de l'objet session.

**Exemple**
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

// Exécution du worker
Worker::runAll();
```

## Vérifier si des données spécifiques de session existent
```php
$session = $request->session();
$has = $session->has('name');
```
Si la session correspondante n'existe pas ou que sa valeur est null, false est retourné, sinon true.

```php
$session = $request->session();
$has = $session->exists('name');
```
Ce code est également utilisé pour vérifier si des données de session existent. La différence est que lorsque la valeur de la session correspondante est nulle, true est renvoyé.


