# En tant que client ws/wss

Parfois, il est nécessaire de permettre à Workerman de fonctionner en tant que client utilisant le protocole ws/wss pour se connecter à un serveur et interagir avec celui-ci. Voici un exemple.

## Workerman en tant que client ws

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // Une fois la poignée de main websocket établie
    $con->onWebSocketConnect = function(AsyncTcpConnection $con, ) {
        $con->send('hello');
    };

    // Lors de la réception d'un message
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman en tant que client wss (ws+ssl)

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Ssl nécessite l'accès au port 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // Configure l'accès en utilisant le chiffrement ssl pour le rendre wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman en tant que client wss (ws+ssl) avec certificat ssl local

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Configure l'accès à l'ip et au port local de l'hôte distant ainsi que le certificat ssl
    $context_option = array(
        // Options ssl, voir http://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Chemin du certificat local. Doit être en format PEM et contenir le certificat local et la clé privée.
            'local_cert'        => '/votre/chemin/vers/le/fichier.pem',
            // Mot de passe du fichier local_cert.
            'passphrase'        => 'votre_mot_de_passe_pem',
            // Autoriser les certificats auto-signés.
            'allow_self_signed' => true,
            // Doit-on vérifier le certificat SSL.
            'verify_peer'       => false
        )
    );

    // Ssl nécessite l'accès au port 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Configure l'accès en utilisant le chiffrement ssl pour le rendre wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Autres configurations
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Lorsque le processus démarre
$worker->onWorkerStart = function()
{
    // Connexion au serveur websocket distant en utilisant le protocole websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Envoyer un ping websocket avec un opcode de 0x9 toutes les 55 secondes au serveur
    $ws_connection->websocketPingInterval = 55;
    // En-têtes HTTP personnalisés
    $ws_connection->headers = ['token' => 'value'];
    // Définir le type de données, par défaut BINARY_TYPE_BLOB pour le texte
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB pour le texte, BINARY_TYPE_ARRAYBUFFER pour les données binaires
    // Une fois la poignée de main TCP achevée
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // Une fois la poignée de main websocket achevée
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Lorsque le serveur websocket distant envoie un message
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // En cas d'erreur de connexion au serveur websocket distant
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // Lorsque la connexion au serveur websocket distant est fermée
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // En cas de fermeture de la connexion, une reconnexion est tentée après 1 seconde
        $connection->reConnect(1);
    };
    // Après configuration des différentes rappels ci-dessus, effectuer la connexion
    $ws_connection->connect();
};
Worker::runAll();
```
