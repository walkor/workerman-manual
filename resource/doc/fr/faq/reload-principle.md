# Principe du redémarrage en douceur
## Qu'est-ce que le redémarrage en douceur ?

Le redémarrage en douceur est différent du redémarrage ordinaire, car il permet de redémarrer le service (généralement pour les services à courte durée de connexion) sans affecter les utilisateurs, afin de recharger le programme PHP et de mettre à jour le code métier.

Le redémarrage en douceur est généralement utilisé lors de mises à jour métier ou de déploiements de versions, afin d'éviter les interruptions temporaires de service dues au redémarrage du service lors de la publication du code.

> **Remarque**
> Le système Windows ne prend pas en charge le rechargement.

> **Remarque**
> Pour les services à longue durée de connexion (comme WebSocket), les connexions seront interrompues lors du redémarrage en douceur. La solution consiste à utiliser une architecture similaire à [gatewayWorker](https://www.workerman.net/doc/gateway-worker), où un groupe de processus maintient spécifiquement les connexions, et la propriété de [reloadable](../worker/reloadable.md) de ce groupe de processus est réglée sur false. La logique métier est gérée par un autre groupe de processus, et la communication entre la passerelle et les processus ouvriers se fait via TCP. Lorsqu'une modification métier est nécessaire, il suffit de redémarrer le processus ouvrier.

## Limitations
**Remarque : Seuls les fichiers chargés dans les rappels on{...} seront automatiquement mis à jour après un redémarrage en douceur. Les fichiers chargés directement dans le script de démarrage ou le code figé ne seront pas mis à jour lors de la reprise.**

#### Le code suivant ne sera pas mis à jour après un redémarrage.
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // Le code figé ne prend pas en charge la mise à jour à chaud
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // Les fichiers chargés directement dans le script de démarrage ne prennent pas en charge la mise à jour à chaud
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // Supposons que la classe MessageHandler contient une méthode onMessage
```

#### Le code suivant sera mis à jour automatiquement après un redémarrage.
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart est le rappel déclenché après le démarrage du processus
    require_once __DIR__ . '/your/path/MessageHandler.php'; // Les fichiers chargés après le démarrage du processus prennent en charge la mise à jour à chaud
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
Après avoir apporté des modifications à MessageHandler.php, exécutez `php start.php reload`, et MessageHandler.php sera rechargé en mémoire pour mettre à jour la logique métier.

> **Conseil**
> Dans le code ci-dessus, pour faciliter la démonstration, la déclaration `require_once` est utilisée. Si votre projet prend en charge le chargement automatique PSR-4, la déclaration `require_once` n'est pas nécessaire.

## Principe du redémarrage en douceur

Workerman se compose du processus principal et des processus fils. Le processus principal est responsable de la surveillance des processus fils, tandis que les processus fils reçoivent les connexions des clients et les demandes de données des connexions, effectuent le traitement approprié et renvoient les données au client. Lors de la mise à jour du code métier, il suffit en réalité de mettre à jour les processus fils.

Lorsque le processus principal de Workerman reçoit un signal de redémarrage en douceur, il envoie un signal de sortie sécurisé (pour que le processus correspondant termine le traitement de la demande en cours avant de sortir). Une fois ce processus sorti, le processus principal crée un nouveau processus fils (qui charge le nouveau code PHP), puis envoie à nouveau une commande d'arrêt à un autre ancien processus, et ainsi de suite, pour remplacer un processus après l'autre, jusqu'à ce que tous les anciens processus soient remplacés.

Nous voyons que le redémarrage en douceur consiste en réalité à faire sortir les anciens processus métier un par un, puis à en créer de nouveaux. Pour garantir qu'il n'y a pas d'impact sur les utilisateurs lors du redémarrage en douceur, il est nécessaire que les processus ne conservent pas d'informations d'état liées aux utilisateurs, c'est-à-dire qu'il est préférable que les processus métier soient sans état, afin d'éviter toute perte d'informations due à la sortie des processus.

