# workerman/mqtt
MQTT est un protocole de transmission de messages en mode publication / abonnement, avec une architecture client-serveur, qui est devenu un élément important de l'Internet des objets. Il est conçu pour être léger, ouvert, simple, normalisé et facile à implémenter. Ces caractéristiques en font un excellent choix pour de nombreux scénarios, en particulier dans des environnements contraints tels que la communication machine à machine (M2M) et l'Internet des objets (IoT).

workerman\mqtt est une bibliothèque cliente MQTT asynchrone basée sur Workerman, qui peut être utilisée pour recevoir ou envoyer des messages selon le protocole MQTT. Elle prend en charge les niveaux de qualité de service `QoS 0`, `QoS 1` et `QoS 2`. Elle prend également en charge les versions `MQTT` `3.1`, `3.1.1` et `5`.

# Adresse du projet
https://github.com/walkor/mqtt

# Installation
````
composer require workerman/mqtt
````

# Exemple
**subscribe.php**
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $mqtt = new Workerman\Mqtt\Client('mqtt://test.mosquitto.org:1883');
    $mqtt->onConnect = function($mqtt) {
        $mqtt->subscribe('test');
    };
    $mqtt->onMessage = function($topic, $content){
        var_dump($topic, $content);
    };
    $mqtt->connect();
};
Worker::runAll();
````
Exécutez en ligne de commande avec ```php subscribe.php start``` pour démarrer.

**publish.php**
````php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $mqtt = new Workerman\Mqtt\Client('mqtt://test.mosquitto.org:1883');
    $mqtt->onConnect = function($mqtt) {
       $mqtt->publish('test', 'hello workerman mqtt');
    };
    $mqtt->connect();
};
Worker::runAll();
````
Exécutez en ligne de commande avec ```php publish.php start``` pour démarrer.

## Interface Client de workerman\mqtt

  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#connect"><code>Client::<b>connect()</b></code></a>
  * <a href="#publish"><code>Client::<b>publish()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>
  * <a href="#disconnect"><code>Client::<b>disconnect()</b></code></a>
  * <a href="#close"><code>Client::<b>close()</b></code></a>
  * <a href="#onConnect"><code>callback <b>onConnect</b></code></a>
  * <a href="#onMessage"><code>callback <b>onMessage</b></code></a>
  * <a href="#onError"><code>callback <b>onError</b></code></a>
  * <a href="#onClose"><code>callback <b>onClose</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

Crée une instance de client MQTT.

  * `$address` L'adresse du serveur MQTT, au format 'mqtt://test.mosquitto.org:1883'.

  * `$options` Un tableau d'options client, pouvant définir les options suivantes :
    * `keepalive` : L'intervalle de temps entre les envois de messages "heartbeat" du client au serveur, par défaut 50 secondes, 0 pour désactiver.
    * `client_id` : L'ID du client, avec une valeur par défaut de ```"workerman-mqtt-client-".mt_rand()```.
    * `protocol_name` : Le nom du protocole, `MQTT` (version 3.1.1) ou `MQIsdp` (version 3.1), par défaut `MQTT`.
    * `protocol_level` : Niveau de protocole, `4` lorsque `protocol_name` est `MQTT`, et `3` lorsque `protocol_name` est `MQIsdp`.
    * `clean_session` : Session propre, par défaut `true`. En le configurant sur `false`, le client peut recevoir des messages hors ligne avec les niveaux de QoS 1 et 2.
    * `reconnect_period` : Intervalle de reconnexion, par défaut `1` seconde, 0 pour désactiver.
    * `connect_timeout` : Délai de connexion MQTT, par défaut 30 secondes.
    * `username` : Nom d'utilisateur, facultatif.
    * `password` : Mot de passe, facultatif.
    * `will` : Message de volonté. Lorsqu'un client se déconnecte, le courtier envoie automatiquement un message de volonté à d'autres clients. Au format :
      * `topic` : Sujet
      * `content` : Contenu
      * `qos` : Niveau de QoS
      * `retain` : Drapeau `retain`
    * `resubscribe`: Lorsque la connexion est interrompue de manière anormale et se reconnecte, réabonner les sujets précédents, par défaut `true`.
    * `bindto`: Spécifie l'adresse IP et le port locaux à utiliser pour la connexion au courtier, valeur par défaut `''`.
    * `ssl` : Option SSL, par défaut `false`. Si défini sur `true`, la connexion SSL est activée. Prend également en charge le passage d'un tableau de contexte SSL pour configurer les certificats locaux, etc. (voir la référence du contexte SSL sur https://php.net/manual/en/context.ssl.php)
    * `debug` : Activer le mode de débogage. En mode débogage, des informations détaillées sur la communication avec le courtier seront affichées, par défaut `false`.

-------------------------------------------------------

<a name="connect"></a>
### connect()

Connecte le client au courtier.

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

Publie un message sur un sujet donné.

* `$topic` : Sujet
* `$content` : Message
* `$options` : Tableau d'options, comprenant
  * `qos` : Niveau de QoS, par défaut `0`
  * `retain` : Retenir le drapeau, par défaut `false`
  * `dup` : Drapeau de redondance, par défaut `false`
* `$callback` - `function (\Exception $exception = null)` (non pris en charge pour les messages QoS 0). Déclenché en cas d'erreur ou de succès lors de la publication. `$exception` est l'objet d'exception, `null` s'il n'y a pas d'erreur, idem ci-dessous.

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

S'abonne à un ou plusieurs sujets.

* `$topic` : String (abonnement à un sujet) ou tableau (abonnement à plusieurs sujets).
Lors de l'abonnement à plusieurs sujets, `$topic` est un tableau avec les sujets comme clés et les niveaux de QoS comme valeurs, par exemple `array('sujet1'=> 0, 'sujet2'=> 1)`.
* `$options` : Tableau d'options d'abonnement, comprenant les paramètres suivants :
  * `qos` : Niveau de QoS, par défaut `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])`
Cette fonction de rappel est déclenchée en cas de succès ou d'erreur lors de l'abonnement.
  * `exception` : Objet d'exception, `null` s'il n'y a pas d'erreur, idem ci-dessus.
  * `granted` : Tableau de résultats d'abonnement, similaire à `array('sujet' => 'qos', 'sujet' => 'qos')` où :
    * `sujet` : Sujet abonné
    * `qos` : Niveau de QoS accepté par le courtier

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

Annule l'abonnement à des sujets.

* `$topic` : String ou tableau de chaînes, par exemple `array('sujet1', 'sujet2')`
* `$callback` - `function (\Exception $e = null)`, déclenché en cas de succès ou d'échec

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

Déconnecte normalement du courtier. Un message `DISCONNECT` est envoyé au courtier.

-------------------------------------------------------

<a name="close"></a>
### close()

Ferme la connexion de force avec le courtier, sans envoyer de message `DISCONNECT`.

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)
Déclenché une fois que la connexion avec le courtier a été établie. À ce stade, le client a reçu le message `CONNACK` du courtier.

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)
`function (sujet, message, paquet) {}`

Déclenchée lorsqu'un client reçoit le message Publish.
* `$topic` : Sujet reçu
* `$content` : Contenu du message reçu
* `$mqtt` : Instance du client MQTT

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)
Déclenchée en cas d'erreur de connexion

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()
Déclenchée lors de la fermeture de la connexion, que ce soit par le client ou par le serveur.
