# Créer un service HTTPS

**Question :**

Comment créer un service HTTPS avec Workerman afin que les clients puissent communiquer via le protocole HTTPS ?

**Réponse :**

Le protocole HTTPS est en fait une combinaison du protocole HTTP et du SSL, c'est-à-dire l'ajout d'une couche SSL au protocole HTTP. Workerman prend en charge le protocole HTTP, ainsi que le SSL (nécessite une version de Workerman >= 3.3.7), il suffit donc d'activer le SSL sur la base du protocole HTTP pour prendre en charge le protocole HTTPS.

Il existe deux solutions courantes pour permettre à Workerman de prendre en charge HTTPS : soit en activant directement le SSL avec Workerman, soit en utilisant Nginx comme proxy SSL. Vous devez choisir l'une ou l'autre de ces solutions, mais pas les deux en même temps.

## Activer le SSL avec Workerman

**Prérequis :**

1. Version de Workerman >= 3.3.7
2. Extension openssl installée pour PHP
3. Certificat SSL déjà obtenu (fichiers pem/crt et clé) disponibles dans /etc/nginx/conf.d/ssl

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Il est préférable d'utiliser un certificat obtenu
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // peut également être un fichier crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // nécessaire pour un certificat auto-signé
    )
);
// Configuration du protocole HTTP
$worker = new Worker('http://0.0.0.0:443', $context);
// Activer le transport SSL pour passer en HTTP+SSL (HTTPS)
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Avec ce code, Workerman crée un service HTTPS, permettant aux clients de se connecter à Workerman de manière sécurisée via le protocole HTTPS.

**Test :**

Saisissez l'URL suivante dans la barre d'adresse du navigateur : `https://domaine:443` pour tester.

**Remarques :**

1. Le port HTTPS doit être accédé en utilisant le protocole HTTPS, et non HTTP.
2. Les certificats sont généralement associés à un domaine, donc utilisez le domaine pour les tests, et non l'adresse IP.
3. En cas d'échec de la connexion HTTPS, veuillez vérifier le pare-feu du serveur.

## Utiliser Nginx comme proxy SSL

En plus d'utiliser le SSL intégré de Workerman, il est également possible d'utiliser Nginx comme proxy SSL pour activer HTTPS.

> **Remarque**
> N'utilisez qu'un seul des deux modes, SSL via Nginx ou SSL via Workerman. Ne les utilisez pas simultanément.

Principe et flux de communication :

1. Le client établit une connexion HTTPS avec Nginx
2. Nginx convertit les données du protocole HTTPS en protocole HTTP et les transmet au port HTTP de Workerman
3. Workerman reçoit les données, les traite pour la logique commerciale, puis renvoie les données au format HTTP à Nginx
4. Nginx reconvertit les données HTTP en HTTPS et les transmet au client

### Configuration Nginx de référence

**Prérequis et préparation :**

1. Supposons que Workerman écoute sur le port 8181 (protocole HTTP).
2. Certificat SSL déjà obtenu (fichiers pem/crt et clé) disponibles dans /etc/nginx/conf.d/ssl
3. Vous désirez utiliser Nginx pour fournir des services de proxy WSS sur le port 443 (le port peut être modifié selon les besoins).

**Exemple de configuration Nginx :**

```conf
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}

server {
  listen 443;
  server_name nom_du_site.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location / {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
