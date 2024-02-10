# Comment envoyer des données à un client spécifique dans WorkerMan

Utiliser un worker pour créer un serveur, sans utiliser GatewayWorker, comment envoyer des messages à des utilisateurs spécifiques ?

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Initialiser un conteneur de travailleurs, écouter le port 1234
$worker = new Worker('websocket://workerman.net:1234');
// ==== Le nombre de processus doit absolument être défini sur 1 ====
$worker->count = 1;
// Ajouter une nouvelle propriété pour enregistrer la correspondance entre uid et la connexion (uid est l'identifiant de l'utilisateur ou l'identifiant unique du client)
$worker->uidConnections = array();
// Callback lorsqu'un client envoie un message
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Vérifier si le client actuel est déjà authentifié, c'est-à-dire si uid est défini
    if(!isset($connection->uid))
    {
       // Si non authentifié, le premier paquet est considéré comme uid (ici, pour une démonstration pratique, l'authentification réelle n'est pas implémentée)
       $connection->uid = $data;
       /* Enregistrer la correspondance entre uid et la connexion, ce qui permet de trouver facilement la connexion par uid,
        * et d'envoyer des données spécifiques à un uid
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('Connexion réussie, votre uid est ' . $connection->uid);
    }
    // Autres logiques, envoi à un uid spécifique ou diffusion générale
    // Supposons que le format du message soit uid: message pour envoyer le message à uid
    // Lorsque uid est "all", c'est une diffusion globale
    list($recv_uid, $message) = explode(':', $data);
    // Diffusion globale
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // Envoyer à un uid spécifique
    else
    {
        sendMessageByUid($recv_uid, $message);
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
    }
}

// Exécuter tous les travailleurs (en réalité, seul un travailleur est défini ici)
Worker::runAll();
```

**Remarque:**

Cet exemple permet d'envoyer des données à un uid spécifique, même s'il est mono-processus, il peut prendre en charge jusqu'à 100 000 utilisateurs en ligne sans problème.

Il est à noter que cet exemple ne peut fonctionner qu'avec un seul processus, c'est-à-dire que $worker->count doit être égal à 1. Pour prendre en charge plusieurs processus ou un cluster de serveurs, il est nécessaire d'utiliser le composant Channel pour la communication inter-processus, le développement est également très simple, vous pouvez consulter la section "Exemple de diffusion en cluster avec le composant Channel" dans la documentation pour plus de détails.

**Si vous souhaitez envoyer des messages à des clients à partir d'autres systèmes, veuillez consulter la section "Envoi de messages dans d'autres projets"**
