# Créer un service wss

**Question :**

Comment créer un service wss avec Workerman, de sorte que les clients puissent se connecter en utilisant le protocole wss, par exemple pour se connecter depuis une mini-application WeChat?

**Réponse:**

Le protocole wss est en réalité [websocket](https://baike.baidu.com/item/WebSocket) + [SSL](https://baike.baidu.com/item/ssl), c'est-à-dire qu'il ajoute une couche [SSL](https://baike.baidu.com/item/ssl) au protocole websocket, similaire à [https](https://baike.baidu.com/item/https) ([http](https://baike.baidu.com/item/http) + [SSL](https://baike.baidu.com/item/ssl)). Par conséquent, il suffit d'activer [SSL](https://baike.baidu.com/item/ssl) sur le protocole [websocket](https://baike.baidu.com/item/WebSocket) pour prendre en charge le protocole wss.

## Méthode 1 : Utilisation de Nginx/Apache pour la représentation SSL (recommandé)

**Principe et processus de communication :**

1. Le client établit une connexion wss vers Nginx/Apache.
2. Nginx/Apache convertit les données du protocole wss en données du protocole ws et les transmet au port du protocole WebSocket de Workerman.
3. Workerman reçoit les données et effectue le traitement logique.
4. Lorsque Workerman envoie un message au client, le processus est inversé : les données sont converties en protocole wss par Nginx/Apache, puis envoyées au client.

## Configuration Nginx

**Prérequis et préparation :**

1. Nginx est installé, version supérieure ou égale à 1.3.
2. Supposons que Workerman écoute sur le port 8282 (protocole websocket).
3. Un certificat (fichier pem/crt et fichier clé) a été demandé et est stocké dans /etc/nginx/conf.d/ssl.
4. Vous prévoyez d'utiliser Nginx pour activer le port 443 en tant que service proxy wss (le port peut être modifié selon vos besoins).
5. Habituellement, Nginx sert à d'autres services web. Pour ne pas interférer avec les sites existants, nous utilisons l'adresse "domain.com/wss" comme point d'entrée de proxy wss. Ainsi, l'adresse de connexion du client sera wss://domain.com/wss.

**Exemple de configuration Nginx :**

```nginx
server {
  listen 443;
  # Configuration de domaine omise...

  ssl on;
  ssl_certificate /etc/ssl/server.pem;
  ssl_certificate_key /etc/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /wss
  {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # Emplacement / {} Autres configurations du site…
}
```

**Test :**

```javascript
// Le certificat vérifie le domaine, veuillez utiliser le nom de domaine pour la connexion. Notez qu'aucun port n'est spécifié ici.
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Connexion établie");
    ws.send('tom');
    alert("Envoi d'une chaîne au serveur : tom");
};
ws.onmessage = function(e) {
    alert("Message reçu du serveur : " + e.data);
};
```

## Utilisation d'Apache pour la représentation wss

Il est également possible d'utiliser Apache comme proxy wss pour transférer vers Workerman.

Prérequis :

1. GatewayWorker écoute sur le port 8282 (protocole websocket).
2. Un certificat SSL a été demandé et est stocké dans /server/httpd/cert/.
3. Apache est configuré pour transférer le port 443 vers le port 8282 spécifié.
4. httpd-ssl.conf est chargé.
5. OpenSSL est installé.

**Activation du module proxy_wstunnel_module :**

```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**Configuration SSL et proxy :**

```apache
# extra/httpd-ssl.conf
DocumentRoot "/chemin/vers/le/site"
ServerName domaine

# Configuration du proxy
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# Ajout de la prise en charge de protocole SSL, suppression des protocoles non sécurisés
SSLProtocol all -SSLv2 -SSLv3
# Modification des suites de chiffrement comme suit
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# Configuration de la clé publique du certificat
SSLCertificateFile /server/httpd/cert/votre.pem
# Configuration de la clé privée du certificat
SSLCertificateKeyFile /server/httpd/cert/votre.key
# Configuration du chaînage de certificats
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**Test :**

```javascript
// Le certificat vérifie le domaine, veuillez utiliser le nom de domaine pour la connexion. Notez qu'aucun port n'est spécifié ici.
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Connexion établie");
    ws.send('tom');
    alert("Envoi d'une chaîne au serveur : tom");
};
ws.onmessage = function(e) {
    alert("Message reçu du serveur : " + e.data);
};
```

## Méthode 2 : Activation directe de SSL avec Workerman (non recommandé)

> **Remarque :**
> L'utilisation de Nginx/Apache pour la représentation SSL et la configuration SSL avec Workerman sont mutuellement exclusives.

**Prérequis :**

1. Version Workerman >= 3.3.7.
2. PHP est installé avec l'extension openssl.
3. Un certificat (fichier pem/crt et clé) est stocké dans un répertoire de votre choix.

**Code :**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Un certificat valide est recommandé
$context = array(
    // Plus d'options SSL sont disponibles dans la documentation http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // Utilisez des chemins absolus
        'local_cert'        => 'chemin/vers/server.pem', // Ou un fichier crt
        'local_pk'          => 'chemin/vers/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Nécessaire pour les certificats autosignés
    )
);
// Ici, le protocole est websocket (le port est arbitraire, mais doit être disponible et non utilisé par d'autres programmes)
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// Configurer le transport pour activer SSL, websocket+SSL signifie wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Avec ce code, Workerman écoute le protocole wss et les clients peuvent se connecter en utilisant le protocole wss pour établir une communication sécurisée en temps réel avec Workerman.

**Test :**

Ouvrez Google Chrome, appuyez sur F12 pour ouvrir la console de débogage, puis saisissez (ou intégrez le code ci-dessous dans une page HTML pour l'exécuter via JavaScript) :

```javascript
// Le certificat vérifie le domaine et doit être utilisé pour la connexion, notez la présence du port ici
ws = new WebSocket("wss://domain.com:8282");
ws.onopen = function() {
    alert("Connexion établie");
    ws.send('tom');
    alert("Envoi d'une chaîne au serveur : tom");
};
ws.onmessage = function(e) {
    alert("Message reçu du serveur : " + e.data);
};
```

**Remarques :**

1. Si vous devez utiliser le port 443, veuillez utiliser la première méthode de représentation SSL avec Nginx/Apache pour implémenter wss.
2. Le port wss ne peut être accessible que via le protocole wss, le protocole ws ne peut pas accéder au port wss.
3. Les certificats sont généralement liés à des noms de domaine, donc pour les tests, veuillez utiliser un nom de domaine pour la connexion et n'utilisez pas l'adresse IP.
4. Si vous rencontrez des problèmes d'accès, veuillez vérifier le pare-feu du serveur.
5. Cette méthode nécessite une version PHP >= 5.6, car les mini-applications WeChat requièrent TLS 1.2, et PHP versions antérieures à 5.6 ne prennent pas en charge TLS 1.2.

Articles connexes :
[Récupération de l'adresse IP réelle du client à travers le proxy](get-real-ip-from-proxy.md)
[Référence des options de contexte SSL pour Workerman](https://php.net/manual/zh/context.ssl.php)
