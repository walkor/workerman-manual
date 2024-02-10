# Transfert crypté - SSL/TLS

**Question :**

Comment garantir la sécurité de la communication avec Workerman ?

**Réponse :**

Une méthode pratique est d'ajouter une couche de chiffrement [SSL](https://fr.wikipedia.org/wiki/Transport_Layer_Security) sur le protocole de communication. Par exemple, les protocoles wss et [https](https://fr.wikipedia.org/wiki/HyperText_Transfer_Protocol_Secure) reposent sur le transfert chiffré par [SSL](https://fr.wikipedia.org/wiki/Transport_Layer_Security) et sont donc très sécurisés. Workerman prend en charge [SSL](https://fr.wikipedia.org/wiki/Transport_Layer_Security) lui-même (nécessite Workerman>=3.3.7), il suffit de configurer les propriétés pour activer [SSL].

Bien entendu, les développeurs peuvent également mettre en place leur propre mécanisme de cryptage en utilisant certains algorithmes de chiffrement.

## Voici la méthode pour activer [SSL] dans Workerman :

**Prérequis :**

1. Version de Workerman supérieure ou égale à 3.3.7
2. Extension openssl installée pour PHP
3. Certificat déjà obtenu (fichier pem/crt et fichier clé) placé dans /etc/nginx/conf.d/ssl

**Code :**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Il est préférable d'avoir un certificat obtenu
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // Peut également être un fichier crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Si c'est un certificat auto-signé, cette option doit être activée
    )
);
// Ici, le protocole utilisé est websocket, mais cela peut être HTTP ou tout autre protocole
$worker = new Worker('websocket://0.0.0.0:443', $context);
// Activer [SSL] pour le transport
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Activer l'indication du nom du serveur [SNI (Server Name Indication)](https://fr.wikipedia.org/wiki/Server_Name_Indication) dans Workerman pour permettre la liaison de plusieurs certificats sur la même IP et port.

**Fusionner les fichiers certificat .pem et .key :**

Combinez le contenu des fichiers .pem et .key de chaque certificat en ajoutant le contenu du fichier .key à la fin du fichier .pem (si le fichier .pem contient déjà la clé privée, cette étape peut être ignorée).

**Veuillez noter qu'il s'agit d'un seul certificat, il ne s'agit pas de copier tous les certificats dans un seul fichier.**

Par exemple, le contenu du fichier pem fusionné après la fusion de *host1.com.pem* ressemblerait à ceci :

```text
-----BEGIN CERTIFICATE-----
MIIGXTCBA...
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIFBzCCA...
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAA....
-----END RSA PRIVATE KEY-----
```

**Code :**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // Activer SNI
        'SNI_server_certs' => [ // Définir plusieurs certificats
            'host1.com' => '/path/host1.com.pem', // Certificat 1
            'host2.com' => '/path/host2.com.pem', // Certificat 2
        ],
        'local_cert' => '/path/default.com.pem', // Certificat par défaut
        'local_pk'   => '/path/default.com.key',
    )
);
$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
