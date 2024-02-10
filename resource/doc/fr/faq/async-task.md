# Comment implémenter des tâches asynchrones

**Question :**

Comment gérer de manière asynchrone des tâches lourdes afin d'éviter le blocage prolongé de la tâche principale ? Par exemple, je dois envoyer des e-mails à 1000 utilisateurs, ce processus est lent et peut bloquer pendant quelques secondes. Pendant ce processus, le blocage de la tâche principale affectera les requêtes ultérieures. Comment déléguer de telles tâches lourdes à d'autres processus de manière asynchrone ?

**Réponse :**

Vous pouvez préalablement mettre en place un certain nombre de processus de tâche pour traiter les tâches lourdes sur votre propre machine, sur un autre serveur, voire sur un cluster de serveurs. Le nombre de processus de tâche peut être augmenté, par exemple jusqu'à 10 fois le nombre de processeurs, puis l'appelant peut utiliser AsyncTcpConnection pour envoyer de manière asynchrone les données à ces processus de tâche afin qu'ils soient traités de manière asynchrone, en recevant ainsi le résultat du traitement de manière asynchrone.

Serveur de processus de tâche
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Travailleur de tâche, utilisant le protocole Texte
$task_worker = new Worker('Text://0.0.0.0:12345');
// Le nombre de processus de tâche peut être augmenté selon vos besoins
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // Supposons que les données reçues sont au format JSON
     $task_data = json_decode($task_data, true);
     // Traitement des tâches en fonction de task_data.... obtention du résultat, ici non détaillé....
     $task_result = ......
     // Envoi du résultat
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Appel dans workerman

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Service websocket
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // Établir une connexion asynchrone avec le service de tâche distant, où l'adresse IP est celle du service de tâche distant, 127.0.0.1 pour une machine locale et l'IP LVS pour un cluster
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // Données de la tâche et ses paramètres
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // Envoi des données
    $task_connection->send(json_encode($task_data));
    // Obtention asynchrone du résultat
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // Résultat
         var_dump($task_result);
         // Après obtention du résultat, n'oubliez pas de fermer la connexion asynchrone
         $task_connection->close();
         // Notification au client websocket correspondant que la tâche est terminée
         $ws_connection->send('tâche terminée');
    };
    // Exécution de la connexion asynchrone
    $task_connection->connect();
};

Worker::runAll();
```

De cette manière, les tâches lourdes sont confiées à des processus sur la machine locale ou sur un autre serveur, et une fois la tâche terminée, le résultat est reçu de manière asynchrone, évitant ainsi le blocage du processus métier.
