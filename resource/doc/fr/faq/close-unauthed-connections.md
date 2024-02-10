# Fermer les connexions non authentifiées

**Question :**

Comment fermer automatiquement la connexion d'un client qui n'envoie pas de données pendant une période spécifiée, par exemple, fermer automatiquement la connexion d'un client qui n'a pas envoyé de données pendant 30 secondes, dans le but de forcer les connexions non authentifiées à s'authentifier dans un délai spécifié ?

**Réponse :**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // Ajouter temporairement une propriété auth_timer_id à l'objet $connection pour stocker l'ID du minuteur
    // Fermer la connexion après 30 secondes par un minuteur, le client doit envoyer une authentification dans les 30 secondes pour supprimer le minuteur
    $connection->auth_timer_id = Timer::add(30, function()use($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ...略
        // Authentification réussie, supprimer le minuteur pour éviter la fermeture de la connexion
        Timer::del($connection->auth_timer_id);
        break;
         ... 略
    }
    ... 略
}
```
