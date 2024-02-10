# Protocole WebSocket

Actuellement, la version du protocole WebSocket dans Workerman est **13**.

Le protocole WebSocket est un nouveau protocole en HTML5. Il permet la communication bidirectionnelle entre le navigateur et le serveur.

## Relation entre WebSocket et TCP

Tout comme HTTP, WebSocket est un protocole de couche application, tous deux étant basés sur la transmission TCP. WebSocket n'a pas beaucoup de liens avec les sockets et encore moins avec eux.

## Poignée de main du protocole WebSocket

Le protocole WebSocket nécessite une poignée de main où le navigateur et le serveur communiquent via le protocole HTTP. Dans Workerman, vous pouvez intervenir ainsi dans le processus de poignée de main.

**Pour workerman <= 4.1**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // Vous pouvez vérifier ici si la source de la connexion est légitime. Si elle ne l'est pas, fermez la connexion
        // $_SERVER['HTTP_ORIGIN'] indique à partir de quelle page du site la connexion WebSocket a été initiée
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // Dans onWebSocketConnect, $_GET et $_SERVER sont accessibles
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**Pour workerman >= 5.0**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## Transmission de données binaires par le protocole WebSocket

Le protocole WebSocket peut par défaut transmettre uniquement des textes UTF-8. Pour transmettre des données binaires, veuillez consulter la section suivante.

Dans le protocole WebSocket, un bit de marquage est utilisé dans l'en-tête du protocole pour indiquer si les données transmises sont des données binaires ou du texte UTF-8. Le navigateur vérifiera si le marquage et le type de contenu transmis sont conformes. Sinon, il se déconnectera avec une erreur.

Ainsi, lors de l'envoi de données côté serveur, il est nécessaire de définir ce bit de marquage en fonction du type de données à transmettre. Dans Workerman, si c'est un texte UTF-8 ordinaire, vous devez définir (c'est la valeur par défaut, donc généralement pas besoin de la définir manuellement)
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

Si ce sont des données binaires, vous devez définir
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**Remarque** : Si $connection->websocketType n'est pas défini, $connection->websocketType sera BINARY_TYPE_BLOB par défaut (c'est-à-dire le type texte UTF-8). En général, les applications transmettent des textes UTF-8, tels que des données JSON, donc il n'est pas nécessaire de définir manuellement $connection->websocketType. Seulement lorsque vous transmettez des données binaires (telles que des données d'image, des données protobuffer, etc.) que vous devez définir cette propriété comme BINARY_TYPE_ARRAYBUFFER.

## Utiliser workerman comme client WebSocket

Vous pouvez utiliser la classe [AsyncTcpConnection](../async-tcp-connection.md) en conjonction avec le protocole [ws](about-ws.md) pour faire de workerman un client WebSocket se connectant à un serveur WebSocket distant, permettant ainsi une communication bidirectionnelle en temps réel.
