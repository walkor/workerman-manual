# onBufferFull
## Description:
```php
callback Worker::$onBufferFull
```

Chaque connexion dispose d'un tampon d'envoi de couche applicative distinct. Si la vitesse de réception du client est inférieure à la vitesse d'envoi du serveur, les données sont temporairement stockées dans le tampon de la couche applicative et si ce tampon est plein, l'événement onBufferFull est déclenché.

La taille du tampon est définie par [TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md), dont la valeur par défaut est de 1 Mo. Il est possible de définir dynamiquement la taille du tampon pour une connexion spécifique, par exemple :
```php
// Définir la taille du tampon d'envoi pour la connexion actuelle, en octets
$connection->maxSendBufferSize = 102400;
```
Il est également possible d'utiliser [TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) pour définir la taille par défaut du tampon pour toutes les connexions, par exemple :
```php
use Workerman\Connection\TcpConnection;
// Définir la taille par défaut du tampon d'envoi de couche applicative pour toutes les connexions, en octets
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```
Ce callback **peut** être déclenché immédiatement après l'appel à Connection::send, par exemple lors de l'envoi de grandes quantités de données ou d'un envoi rapide et continu de données à la partie distante. Cela se produit lorsque, pour diverses raisons telles que le réseau, les données s'accumulent considérablement dans le tampon d'envoi de la connexion correspondante, dépassant ainsi la limite de ```TcpConnection::$maxSendBufferSize```.

Lorsque l'événement onBufferFull se produit, les développeurs doivent généralement prendre des mesures, telles que cesser d'envoyer des données à la partie distante et attendre que les données du tampon d'envoi soient entièrement envoyées (événement onBufferDrain).

Lorsque l'appel à Connection::send(`$A`) déclenche onBufferFull, peu importe la taille des données `$A` envoyées cette fois-ci, même si elles dépassent la valeur de `TcpConnection::$maxSendBufferSize`, les données à envoyer cette fois-ci seront quand même placées dans le tampon d'envoi. Cela signifie que la taille réelle des données placées dans le tampon d'envoi peut largement dépasser `TcpConnection::$maxSendBufferSize`. Lorsque la taille du tampon d'envoi dépasse déjà `TcpConnection::$maxSendBufferSize`, un nouvel appel à Connection::send(`$B`) sera rejeté et les données `$B` seront ignorées, déclenchant ainsi le callback `onError`.

En résumé, tant que le tampon d'envoi n'est pas plein, même s'il ne reste qu'un octet d'espace, l'appel à Connection::send(`$A`) placera certainement `$A` dans le tampon d'envoi. Si, après que les données ont été placées dans le tampon d'envoi, la taille du tampon dépasse la limite de `TcpConnection::$maxSendBufferSize`, l'événement onBufferFull sera déclenché.

## Callback Function Parameters

``` $connection ```

L'objet de connexion, c'est-à-dire une instance de [TcpConnection](../tcp-connection.md) utilisée pour manipuler la connexion client, tels que [l'envoi de données](../tcp-connection/send.md), [la fermeture de la connexion](../tcp-connection/close.md), etc.

## Example
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "Le tampon est plein et l'envoi est interrompu\n";
};
// Exécute le worker
Worker::runAll();
```
Remarque : En plus d'utiliser une fonction anonyme comme rappel, il est également possible de [consulter ici](../faq/callback_methods.md) d'autres méthodes de rappel.

## Voir aussi
onBufferDrain : Déclenché lorsqu'il ne reste plus aucune donnée dans le tampon d'envoi de la couche applicative de la connexion.
