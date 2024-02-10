# send
## Description:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

Envoyer des données au client

## Parameters

 ``` $data ```
 
Les données à envoyer. Si un protocole est spécifié lors de l'initialisation de la classe Worker, la méthode d'encodage du protocole sera automatiquement appelée pour empaqueter les données selon le protocole avant de les envoyer au client.

 ``` $raw ```
 
Indique si les données brutes doivent être envoyées, c'est-à-dire sans utiliser la méthode d'encodage du protocole. Par défaut, c'est false, ce qui signifie que la méthode d'encodage du protocole est automatiquement utilisée.

## Return Value

true : Les données ont été écrites avec succès dans le tampon d'envoi du socket de cette connexion au niveau du système d'exploitation.

null : Les données ont été écrites dans le tampon d'envoi de la couche d'application de cette connexion, en attente d'être écrites dans le tampon d'envoi du socket au niveau du système d'exploitation.

false : L'envoi a échoué, peut-être parce que la connexion client est déjà fermée ou que le tampon d'envoi de la couche d'application de cette connexion est plein.

## Note
Lorsque send renvoie ```true```, cela signifie seulement que les données ont été écrites avec succès dans le tampon d'envoi du socket de cette connexion au niveau du système d'exploitation, mais cela ne garantit pas que les données ont été effectivement envoyées au tampon de réception du socket distant, et encore moins que l'application du côté distant a déjà lu les données du tampon de réception du socket local. **Cependant, tant que send ne renvoie pas false et que la connexion réseau est active et que le client reçoit normalement, on peut considérer que les données peuvent être transmises à 100% de manière fiable au destinataire.**

Étant donné que les données du tampon d'envoi du socket sont envoyées de manière asynchrone à distance par le système d'exploitation, et que le système d'exploitation ne fournit pas de mécanisme de confirmation à l'application, **l'application n'a pas la possibilité de savoir quand les données du tampon d'envoi du socket commencent à être envoyées, et encore moins de savoir si l'envoi des données du tampon d'envoi du socket a réussi. C'est la raison pour laquelle workerman ne peut pas fournir directement d'interface de confirmation des messages.**

Si l'activité nécessite de garantir que chaque message est bien reçu par le client, il est possible d'ajouter un mécanisme de confirmation dans l'application. Ce mécanisme de confirmation peut varier en fonction des besoins métier, et même pour un même cas d'utilisation, plusieurs méthodes de confirmation peuvent être envisagées.

Par exemple, pour un système de messagerie, on peut utiliser le mécanisme de confirmation suivant : chaque message est stocké dans la base de données avec un indicateur spécifiant s'il a été lu ou non. Lorsqu'un client reçoit un message, il envoie un paquet de confirmation au serveur, qui marque alors le message correspondant comme lu. Lorsqu'un client se connecte au serveur (généralement lorsqu'il se connecte ou se reconnecte après une perte de connexion), le serveur vérifie la base de données pour voir s'il y a des messages non lus, et les envoie au client. De la même manière, lorsque le client reçoit un message, il informe le serveur qu'il l'a lu. De cette façon, on peut garantir que chaque message est bien reçu par le destinataire. Bien sûr, les développeurs peuvent également utiliser leur propre logique de confirmation selon leurs besoins.

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // La méthode d'encodage \Workerman\Protocols\Websocket::encode est automatiquement appelée pour empaqueter les données selon le protocole Websocket avant de les envoyer
    $connection->send("hello\n");
};
// Exécute le worker
Worker::runAll();
```
