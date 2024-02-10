# écouter
```php
void Worker::listen(void)
```
Utilisé pour instancier Worker et écouter. Cette méthode est principalement utilisée pour créer dynamiquement de nouvelles instances de Worker après le démarrage du processus Worker. Cela permet à un même processus d'écouter plusieurs ports et de supporter différents protocoles. Il est important de noter que l'utilisation de cette méthode n'ajoute qu'une écoute au processus actuel, sans créer dynamiquement de nouveaux processus ni déclencher la méthode onWorkerStart.

Par exemple, une fois qu'un Worker HTTP est démarré, une autre instance de Worker websocket peut être instanciée, permettant ainsi au processus de répondre à la fois aux requêtes HTTP et aux connexions websocket. Étant donné que les Workers websocket et HTTP partagent le même processus, ils peuvent accéder à des variables mémoire communes et partager toutes les connexions socket, ce qui permet de recevoir une requête HTTP, puis de manipuler le client websocket pour envoyer des données aux clients.

**Remarque :**

Si la version de PHP est inférieure ou égale à 7.0, il n'est pas possible d'instancier la même instance de Worker sur le même port dans plusieurs sous-processus. Par exemple, si le processus A crée un Worker écoutant sur le port 2016, le processus B ne pourra pas créer un Worker écoutant également sur le port 2016, sinon une erreur "Address already in use" sera déclenchée. Le code suivant ne pourra pas fonctionner :

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4 processus
$worker->count = 4;
// Chaque processus démarre une nouvelle instance de Worker pour une écoute actuelle
$worker->onWorkerStart = function($worker)
{
    /**
     * Les 4 processus démarrant créeront un Worker écoutant sur le port 2016
     * Lors de l'exécution de worker->listen(), une erreur "Address already in use" sera déclenchée
     * Si worker->count=1, aucune erreur ne sera déclenchée
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // Écouter. Ici, une erreur "Address already in use" sera déclenchée
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Exécuter le Worker
Worker::runAll();
```

Si votre version de PHP est supérieure ou égale à 7.0, vous pouvez définir Worker->reusePort=true. Cela permet de créer plusieurs sous-processus écoutant sur le même port pour un même Worker. Voici un exemple :

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4 processus
$worker->count = 4;
// Chaque processus démarre une nouvelle instance de Worker pour une écoute actuelle
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // Activer la réutilisation du port pour permettre la création de Workers écoutant sur le même port (nécessite PHP>=7.0)
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Écouter. L'écoute normale ne déclenchera pas d'erreur
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Exécuter le Worker
Worker::runAll();
```


### Exemple de push de messages en temps réel depuis le back-end PHP vers le client

**Principe :**

1. Mettre en place un Worker websocket pour maintenir une connexion longue avec le client
2. Créer un Worker text à l'intérieur du Worker websocket
3. Le Worker websocket et le Worker text sont dans le même processus, ce qui permet de partager facilement les connexions clients
4. Un système PHP backend distinct communique avec le Worker text via le protocole text
5. Le Worker text opère la connexion websocket pour effectuer l'envoi de données

**Code et étapes**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Initialiser un conteneur de workers, écouter le port 1234
$worker = new Worker('websocket://0.0.0.0:1234');

/**
 * Remarque : le nombre de processus doit être défini à 1
 */
$worker->count = 1;
// Lorsque le processus Worker démarre, créer un Worker text pour ouvrir un port de communication interne
$worker->onWorkerStart = function($worker)
{
    // Ouvrir un port interne pour faciliter l'envoi de données par le système interne, format de protocole Text: texte + saut de ligne
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // Le tableau $data contient l'uid, indiquant vers quelle uid les données doivent être poussées
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // Via workerman, envoyer des données à la page de l'uid
        $ret = sendMessageByUid($uid, $buffer);
        // Retourner le résultat de l'envoi
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## Écouter ##
    $inner_text_worker->listen();
};
// Ajouter une nouvelle propriété pour mapper uid aux connexions
$worker->uidConnections = array();
// Lorsqu'un client envoie un message
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Vérifier si le client a déjà été authentifié, c'est-à-dire s'il a défini un uid
    if(!isset($connection->uid))
    {
       // Si non authentifié, le premier paquet est considéré comme un uid (ici, pour une démonstration facile, une véritable vérification n'a pas été effectuée)
       $connection->uid = $data;
       /* Mapper l'uid à la connexion. Cela permet de retrouver facilement la connexion par uid et d'envoyer des données spécifiques à cet uid */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// Lorsqu'un client se déconnecte
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Supprimer la correspondance lors de la déconnexion
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Envoyer des données à tous les utilisateurs authentifiés
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Envoyer des données à un uid spécifique
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// Exécuter tous les workers
Worker::runAll();
```       

Démarrer le service back-end
```php push.php start -d```

Code JS pour recevoir les messages push côté client
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

Code pour pousser les messages depuis le back-end
```php
// Établir une connexion par socket au port de communication interne
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// Données à pousser, contenant le champ uid pour spécifier à quel uid les données doivent être poussées
$data = array('uid'=>'uid1', 'percent'=>'88%');
// Envoyer les données, noter que le port 5678 est un port de protocole Text et nécessite un saut de ligne à la fin des données
fwrite($client, json_encode($data)."\n");
// Lire le résultat de l'envoi
echo fread($client, 8192);
```
