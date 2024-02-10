```php
# on
**``` (La version de Workerman doit être >= 3.3.0) ```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
S'abonner à l'événement ```$event_name``` et enregistrer la fonction de rappel ```$callback_function``` lorsque l'événement se produit.

## Paramètres de la fonction de rappel

``` $event_name ```

Le nom de l'événement auquel s'abonner, qui peut être n'importe quelle chaîne de caractères.

``` $callback_function ```

La fonction de rappel déclenchée lorsque l'événement se produit. Le prototype de la fonction est ```callback_function(mixed $event_data)```. ```$event_data``` est la donnée de l'événement transmise lors de sa publication.

Remarque:

Si deux fonctions de rappel sont enregistrées pour le même événement, la deuxième fonction de rappel remplacera la première.

## Exemple
Worker multi-processus (multi-serveurs), un client envoie un message qui est diffusé à tous les clients

start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Initialiser un serveur Channel
$channel_server = new Channel\Server('0.0.0.0', 2206);

// Serveur websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// Au démarrage de chaque processus worker
$worker->onWorkerStart = function($worker)
{
    // Le client Channel se connecte au serveur Channel
    Channel\Client::connect('127.0.0.1', 2206);
    // S'abonner à l'événement de diffusion (broadcast) et enregistrer la fonction de rappel de l'événement
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // Diffuser le message à tous les clients du processus worker actuel
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // Utiliser les données envoyées par le client comme données de l'événement
   $event_data = $data;
   // Publier l'événement de diffusion (broadcast) à tous les processus worker
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**Test**

Ouvrez le navigateur Chrome, appuyez sur F12 pour ouvrir la console de débogage, dans la section Console, saisissez (ou placez le code ci-dessous dans une page HTML et exécutez-le avec JavaScript)

Connexion pour recevoir les messages
```javascript
// Remplacer 127.0.0.1 par l'adresse IP réelle où se trouve Workerman
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("Message reçu du serveur : " + e.data);
};
```

Diffuser un message
```
ws.send('bonjour tout le monde');
```
