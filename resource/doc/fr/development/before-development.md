# Lecture obligatoire avant le développement

Pour développer une application avec Workerman, vous devez comprendre les éléments suivants :

## I. Différences entre le développement avec Workerman et le développement PHP standard

Mis à part l'impossibilité d'utiliser directement les fonctions et variables liées au protocole HTTP, le développement avec Workerman ne diffère pas beaucoup du développement PHP standard.

### 1. Différences de protocoles d'application
* Le développement PHP standard est généralement basé sur le protocole d'application HTTP, où le serveur Web se charge de l'analyse du protocole.
* Workerman prend en charge divers protocoles. Actuellement, il inclut les protocoles HTTP et WebSocket. Workerman recommande aux développeurs d'utiliser des protocoles de communication personnalisés plus simples.

  * Pour le développement basé sur le protocole HTTP, veuillez consulter la section [Service HTTP](../http/request.md).

### 2. Différences de cycle de requête
* En PHP, après une requête dans une application Web, toutes les variables et ressources sont libérées.
* Les applications développées avec Workerman restent en mémoire après leur chargement initial, ce qui permet aux définitions de classe, aux objets globaux et aux membres statiques de classe de ne pas être libérés, facilitant leur réutilisation ultérieure.

### 3. Éviter les définitions de classe et de constantes en double
* Étant donné que Workerman met en cache les fichiers PHP compilés, il est conseillé d'éviter d'inclure plusieurs fois les mêmes fichiers définissant des classes ou des constantes. Il est recommandé d'utiliser require_once/include_once pour charger les fichiers.

### 4. Libération des ressources de connexion dans le mode singleton
* Étant donné que Workerman ne libère pas les objets globaux et les membres statiques de classe après chaque requête, dans les cas de mode singleton comme les bases de données, l'instance de base de données (qui contient une connexion de socket à la base de données) est souvent conservée dans un membre statique de la base de données. Ainsi, Workerman réutilise cette connexion de socket à la base de données pendant le cycle de vie du processus. Il convient de noter que si le serveur de base de données détecte une inactivité prolongée de la connexion, il peut fermer activement la connexion de socket. Dans ce cas, réutiliser cette instance de base de données provoquera une erreur (semblable à "mysql gone away"). Workerman fournit une [classe de base de données](../components/workerman-mysql.md) avec une fonctionnalité de reconnexion, que les développeurs peuvent utiliser directement.

### 5. Éviter d'utiliser les instructions exit et die
* Workerman fonctionne en mode console PHP. Lors de l'appel aux instructions exit et die, le processus en cours est interrompu. Bien que le sous-processus soit immédiatement remplacé après sa fermeture, cela pourrait tout de même avoir un impact sur l'activité commerciale.

### 6. Redémarrage nécessaire du service après modification du code
Comme Workerman est en mémoire permanente, une fois que les classes et les fonctions PHP sont chargées, elles restent en mémoire et ne sont pas lues à partir du disque. Par conséquent, toute modification du code métier nécessite un redémarrage pour prendre effet.

## II. Concepts de base à connaître

### 1. Protocole de couche de transport TCP
TCP est un protocole de transmission fiable basé sur IP et orienté connexion. Une caractéristique importante du protocole TCP est qu'il est basé sur un flux de données, de sorte que les requêtes des clients sont envoyées en continu au serveur, et le serveur peut recevoir des données qui ne constituent pas une requête complète ou qui pourraient contenir plusieurs requêtes concaténées. Cela nécessite la définition de règles pour distinguer les frontières de chaque requête dans ce flux de données. Les protocoles de la couche d'application sont principalement destinés à définir des règles pour délimiter les données de requête et éviter leur confusion.

### 2. Protocole de couche d'application
Un protocole de couche d'application (application layer protocol) définit comment les processus des applications exécutées sur différents systèmes terminaux (client, serveur) se transmettent des messages. Par exemple, HTTP et WebSocket font partie des protocoles de couche d'application. Un exemple simple de protocole de couche d'application serait : ```{"module":"user","action":"getInfo","uid":456}\n```. Ce protocole utilise ```"\n"``` (remarquez que  ```"\n"``` représente un retour à la ligne) pour marquer la fin de la requête, avec le corps du message étant une chaîne de caractères.

### 3. Connexion à courte durée
Une connexion à courte durée est établie lorsque les deux parties communiquent et se déconnectent une fois l'envoi de données terminé, avec chaque connexion ne gérant qu'une seule tâche de transaction. Les services HTTP des sites Web, par exemple, utilisent généralement des connexions à courte durée.

### 4. Connexion à longue durée
Une connexion à longue durée permet l'envoi de plusieurs paquets de données sur une seule connexion. Il est important de noter que les applications avec des connexions à longue durée doivent avoir un [mécanisme de battements de cœur](../faq/heartbeat.md), sinon la connexion risque d'être coupée par le pare-feu du nœud de routage en raison d'une inactivité prolongée.

Les connexions à longue durée sont couramment utilisées dans les cas de communication fréquente et peer-to-peer. Chaque connexion TCP nécessite une étape de trois phases pour sa mise en place, ce qui prend du temps. Si chaque opération commence par une connexion, puis par une opération, cela ralentira considérablement le traitement. Par conséquent, avec une connexion à longue durée, la connexion n'est pas fermée après chaque opération, ce qui permet d'envoyer directement le paquet de données lors de la prochaine opération, sans établir de nouvelle connexion. Par exemple, les connexions à base de données utilisent une connexion à longue durée. Des communications fréquentes avec des connexions à courte durée peuvent entraîner des erreurs de socket, et la création fréquente de sockets consomme des ressources.

*Les applications avec des connexions à longue durée peuvent se référer au processus de développement Gateway/Worker.*

### 5. Rechargement en douceur
Un redémarrage normal implique l'arrêt de tous les processus, suivis de la création de nouveaux processus. Pendant cette période, il n'y a aucun service disponible, ce qui peut entraîner des échecs de requêtes en cas de forte charge.

Le rechargement en douceur, quant à lui, consiste à ne pas arrêter tous les processus en une seule fois, mais à les remplacer un par un. Chaque fois qu'un processus est arrêté, il est immédiatement remplacé par un nouveau processus, jusqu'à ce que tous les anciens processus soient remplacés.

Avec Workerman, vous pouvez utiliser la commande ```php your_file.php reload``` pour mettre à jour l'application sans affecter la qualité du service.

**Remarque : seuls les fichiers chargés automatiquement dans les rappels on{...} seront mis à jour automatiquement après un rechargement en douceur. Les fichiers chargés directement dans le script de démarrage ou le code incrusté ne seront pas mis à jour automatiquement.**

## III. Distinction entre le processus principal et les sous-processus
Il est important de savoir si le code s'exécute dans le processus principal ou dans un sous-processus. Généralement, tout ce qui est exécuté avant l'appel à ```Worker::runAll();``` s'exécute dans le processus principal, tandis que le code exécuté dans les rappels onXXX est exécuté dans un sous-processus. Il convient de noter que le code situé après ```Worker::runAll();``` ne sera jamais exécuté.

Par exemple, dans le code suivant :
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Exécuté dans le processus principal
$tcp_worker = new Worker("tcp://0.0.0.0:2347");
// L'assignation est exécutée dans le processus principal
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Cette partie s'exécute dans un sous-processus
    $connection->send('hello ' . $data);
};

Worker::runAll();
```

**Remarque :** N'initialisez pas les connexions de base de données, de memcache ou de redis dans le processus principal, car les connexions initialement créées dans le processus principal peuvent être héritées automatiquement par les sous-processus (en particulier avec l'utilisation de singletons). Dans ce cas, tous les processus partagent la même connexion, ce qui peut entraîner la confusion des données. De même, la fermeture de la connexion par l'un des processus (par exemple, lorsque le processus principal se ferme lors de l'exécution en mode daemon) entraînera la fermeture de toutes les connexions des sous-processus en même temps, causant des erreurs imprévues, comme des erreurs "mysql gone away".

Il est recommandé d'initialiser les ressources de connexion dans le rappel onWorkerStart.


