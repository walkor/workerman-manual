# Exemple 1
**``` (Workerman version requise >=3.3.0) ```**

Système de diffusion basé sur les processus multiples (cluster distribué) basé sur les travailleurs, diffusion en cluster, diffusion en cluster.

`start_channel.php`
Un seul service start_channel peut être déployé dans tout le système. Supposons que cela fonctionne sur 192.168.1.1.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Initialiser un serveur de canal
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
Plusieurs services de démarrage ws peuvent être déployés dans tout le système, supposons qu'ils fonctionnent sur 192.168.1.2 et 192.168.1.3 deux serveurs.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Serveur websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Connexion du client du canal au serveur du canal
    Channel\Client::connect('192.168.1.1', 2206);
    // En utilisant l'ID de son propre processus comme nom d'événement
    $event_name = $worker->id;
    // Abonnement à l'événement worker->id et enregistrement de la fonction de gestion des événements
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "la connexion n'existe pas\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // Abonnement à l'événement de diffusion
    $event_name = 'Diffusion';
    // Lorsqu'un événement de diffusion est reçu, envoyer des données de diffusion à toutes les connexions client dans le processus actuel
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "ID du travailleur : {$worker->id} ID de connexion : {$connection->id} connecté\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
Plusieurs services de démarrage http peuvent être déployés dans tout le système, supposons qu'ils fonctionnent sur 192.168.1.4 et 192.168.1.5 deux serveurs.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Utilisé pour traiter les requêtes http, envoyer des données à n'importe quelle connexion client, nécessite workerID et connectionID
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'éditeur';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // Compatible workerman4.x
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // C'est pour pousser des données vers un certain processus worker et une connexion
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // C'est pour une diffusion globale de données
    else
    {
        $event_name = 'Diffusion';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```

## Test
1. Exécutez les services sur les différents serveurs.

2. Connexion du client au serveur

Ouvrez le navigateur Chrome, appuyez sur F12 pour ouvrir la console de débogage, saisissez (ou insérez le code ci-dessous dans une page HTML et exécutez-le en utilisant JavaScript)

```javascript
// Vous pouvez également vous connecter à ws://192.168.1.3:4236
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("Message reçu du serveur : " + e.data);
};
```

3. Pousser en utilisant l'API http

Visitez l'URL ```http://192.168.1.4:4237/?content={$content}``` ou ```http://192.168.1.5:4237/?content={$content}``` pour pousser les données ```$content``` à toutes les connexions client.

Visitez l'URL ```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` ou ```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` pour pousser les données ```$content``` à une connexion client spécifique dans un processus worker spécifique.

Remarque : lors des tests, remplacez ```{$worker_id}``` ```{$connection_id}``` et ```{$content}``` par les valeurs réelles.
