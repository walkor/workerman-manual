# Protocole ws

La version actuelle du protocole ws pour Workerman est la **version 13**.

Workerman peut être utilisé en tant que client, pour établir une connexion websocket via le protocole ws, afin de se connecter à un serveur websocket distant et ainsi permettre une communication bidirectionnelle.

> **Remarque**
> Le protocole ws ne peut être utilisé que via AsyncTcpConnection en tant que client, et ne peut pas être utilisé en tant que protocole d'écoute du serveur websocket. Autrement dit, la syntaxe suivante est incorrecte :

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

Si vous souhaitez utiliser Workerman en tant que serveur websocket, veuillez utiliser le [protocole websocket](about-websocket.md).

**Exemple d'utilisation du protocole ws en tant que client websocket:**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Au démarrage du processus
$worker->onWorkerStart = function()
{
    // Connexion au serveur websocket distant avec le protocole websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Envoi optionnel d'un heartbeat websocket avec un opcode 0x9 toutes les 55 secondes vers le serveur (optionnel)
    $ws_connection->websocketPingInterval = 55;
    // Configuration des en-têtes HTTP (optionnel)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // Configuration du type de données (optionnel)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB pour le texte, BINARY_TYPE_ARRAYBUFFER pour le binaire
    // Une fois la connexion TCP établie (optionnel)
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // Une fois la poignée de main websocket établie (optionnel)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Lorsque le serveur websocket distant envoie un message
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // En cas d'erreur de la connexion, généralement une erreur de connexion au serveur websocket distant (optionnel)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // Lorsque la connexion au serveur websocket distant est close (optionnel, recommencer la connexion est recommandé)
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // En cas de fermeture de la connexion, reconnexion une seconde plus tard
        $connection->reConnect(1);
    };
    // Une fois que tous ces rappels sont configurés, effectuez l'opération de connexion
    $ws_connection->connect();
};
Worker::runAll();
```

Pour plus d'informations, consultez [Utilisation en tant que client ws/wss](../faq/as-wss-client.md).
