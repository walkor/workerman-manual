# Battement de cœur

Remarque : les applications à connexion longue doivent être équipées d'un battement de cœur, sinon la connexion risque d'être interrompue de force par un nœud de routage en raison d'une absence de communication pendant une longue période. 

Le battement de cœur a principalement deux fonctions : 

1. Le client envoie périodiquement des données au serveur pour empêcher la fermeture de la connexion en raison de l'absence de communication pendant une longue période, provoquée par les pare-feu de certains nœuds.
   
2. Le serveur peut déterminer si le client est en ligne en utilisant le battement de cœur. Si le client n'envoie aucune donnée dans un délai spécifié, le serveur considère que le client est hors ligne. Cela permet de détecter les événements où le client se déconnecte en raison de situations extrêmes (coupure de courant, perte de réseau, etc.).

Intervalles de battement de cœur recommandés :

Il est recommandé que le client envoie un battement de cœur à des intervalles inférieurs à 60 secondes, par exemple 55 secondes.

> Le format des données de battement de cœur n'est pas spécifié, tant que le serveur peut les reconnaître.

## Exemple de battement de cœur
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Intervalle de battement de cœur de 55 secondes
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // Définit temporairement une propriété lastMessageTime de la connexion pour enregistrer le moment de la dernière réception du message
    $connection->lastMessageTime = time();
    // Autres logiques métier...
};

// Après le démarrage du processus, configure un minuteur pour s'exécuter toutes les 10 secondes
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function()use($worker){
        $time_now = time();
        foreach($worker->connections as $connection) {
            // Il se peut que cette connexion n'ait pas encore reçu de message, donc lastMessageTime est défini sur l'heure actuelle
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // Si l'intervalle de temps depuis la dernière communication est supérieur à l'intervalle de battement de cœur, il est considéré que le client s'est déconnecté, donc fermeture de la connexion
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

La configuration ci-dessus indique que si le client ne transmet aucune donnée au serveur pendant plus de 55 secondes, le serveur considère que le client s'est déconnecté, ferme la connexion et déclenche onClose.

## Reconnexion en cas de déconnexion (Important)

Que ce soit le client ou le serveur qui envoie un battement de cœur, la connexion peut être interrompue. Par exemple, lorsque le navigateur est réduit au minimum, le JavaScript est suspendu, lorsque le navigateur est basculé sur d'autres onglets, le JavaScript est suspendu, lorsque l'ordinateur passe en mode veille, etc. Sur mobile, il peut y avoir des changements de réseau, une diminution du signal, l'extinction de l'écran, le basculement de l'application en arrière-plan, des défaillances du routeur, la déconnexion volontaire du client, etc. Surtout dans des environnements externes complexes, de nombreux nœuds de routage nettoient les connexions inactives depuis plus d'une minute, c'est pourquoi l'intervalle de battement de cœur est recommandé comme étant inférieur à 1 minute.

Les connexions sont facilement interrompues dans des environnements externes, donc la reconnexion en cas de déconnexion est une fonctionnalité indispensable pour les applications à connexion longue (la reconnexion en cas de déconnexion ne peut être réalisée que côté client, le serveur ne peut pas le faire). Par exemple, les WebSockets du navigateur doivent écouter l'événement onclose, et lorsqu'il se produit, établir une nouvelle connexion (pour éviter les effondrements, cette action doit être retardée). Plus strictement, le serveur devrait également envoyer périodiquement des battements de cœur, et le client doit surveiller régulièrement si les données de battement de cœur du serveur sont en retard. Si aucune donnée de battement de cœur n'est reçue dans le délai spécifié, la connexion est considérée comme étant interrompue, et doit donc être close, puis une nouvelle connexion doit être établie.
