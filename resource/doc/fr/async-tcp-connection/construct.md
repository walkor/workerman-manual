# Méthode __construct
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
Crée un objet de connexion asynchrone.

AsyncTcpConnection permet à Workerman d'agir en tant que client pour établir une connexion asynchrone avec un serveur distant, et d'envoyer et de recevoir des données de manière asynchrone via l'interface send et le rappel onMessage.

## Paramètres
Paramètre :```remote_address```

L'adresse de la connexion, par exemple
 ``` tcp://www.baidu.com:80 ```
 ``` ssl://www.baidu.com:443 ```
 ``` ws://echo.websocket.org:80 ```
 ``` frame://192.168.1.1:8080 ```
 ``` text://192.168.1.1:8080 ```

Paramètre :```$context_option```
 ```Cette valeur est requise (workerman >= 3.3.5)```

Utilisé pour définir le contexte du socket, par exemple en utilisant ```bindto``` pour définir à partir de quelle adresse IP et port (carte réseau) accéder au réseau externe, définir des certificats SSL, etc.

Référez-vous à [stream_context_create](https://php.net/manual/en/function.stream-context-create.php), [Options de contexte du socket](https://php.net/manual/zh/context.socket.php), [Options de contexte SSL](https://php.net/manual/zh/context.ssl.php)

## Remarque
Actuellement, AsyncTcpConnection prend en charge les protocoles suivants : [tcp](https://baike.baidu.com/subview/32754/8048820.htm), [ssl](https://baike.baidu.com/view/525499.htm), [ws](appendices/about-ws.md), [frame](appendices/about-frame.md), [text](appendices/about-text.md).

Il prend également en charge les protocoles personnalisés, voyez [Comment personnaliser les protocoles](../protocols/how-protocols.md).

Le protocole [ssl](https://baike.baidu.com/view/525499.htm) nécessite Workerman>=3.3.4 et l'installation de l'extension [openssl](https://php.net/manual/zh/book.openssl.php).

Actuellement, AsyncTcpConnection ne prend pas en charge le protocole [http](https://baike.baidu.com/view/9472.htm).

Vous pouvez utiliser ```new AsyncTcpConnection('ws://...')``` pour initier une connexion websocket distante dans Workerman, tout comme dans un navigateur, voir [Exemple](../appendices/about-ws.md). Cependant, il n'est pas possible d'initier une connexion websocket dans Workerman de la forme ```new AsyncTcpConnection('websocket://...')```.

## Exemples
### Exemple 1: Accès asynchrone à un service http externe
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Au démarrage du processus, établir de manière asynchrone une connexion à www.baidu.com et envoyer des données pour obtenir une réponse
$task->onWorkerStart = function($task)
{
    // Ne prend pas en charge une spécification directe d'HTTP, mais peut envoyer des données simulées en utilisant le protocole TCP
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // Lorsque la connexion est établie avec succès, envoyer des données de requête HTTP
    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Connexion établie avec succès\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Connexion fermée\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Code d'erreur:$code msg:$msg\n";
    };
    $connection_to_baidu->connect();
};

// Exécution du worker
Worker::runAll();
```

### Exemple 2: Accès asynchrone à un service websocket externe et configuration de l'adresse IP locale et du port d'accès
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Définir l'adresse IP locale et le port d'accès au serveur distant (chaque connexion socket occupera un port local)
    $context_option = array(
        'socket' => array(
            // L'adresse IP doit être l'adresse IP de la carte réseau locale et doit pouvoir accéder au serveur distant, sinon elle sera invalide
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('Bonjour');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

### Exemple 3: Accès asynchrone à un port wss externe et configuration d'un certificat SSL local
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Définir l'adresse IP locale et le port d'accès au serveur distant ainsi que le certificat SSL
    $context_option = array(
        'socket' => array(
            // L'adresse IP doit être l'adresse IP de la carte réseau locale et doit pouvoir accéder au serveur distant, sinon elle sera invalide
            'bindto' => '114.215.84.87:2333',
        ),
        // Option SSL, consultez https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Chemin du certificat local. Doit être au format PEM et contenir le certificat local et la clé privée.
            'local_cert'        => '/votre/chemin/vers/fichier.pem',
            // Mot de passe du fichier local_cert
            'passphrase'        => 'votre_phrase_de_passe_pem',
            // Autoriser les certificats auto-signés.
            'allow_self_signed' => true,
            // Vérifier si le certificat SSL est requis.
            'verify_peer'       => false
        )
    );

    // Initié une connexion asynchrone
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Définir un accès chiffré SSL
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('Bonjour');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
