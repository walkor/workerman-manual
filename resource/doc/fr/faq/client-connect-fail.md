# Raisons de l'échec de la connexion du client

En général, lorsqu'une connexion échoue, le client peut rencontrer deux types d'erreurs : ```refus de connexion``` (connection refuse) et ```dépassement du délai de connexion``` (connection timeout).

## Refus de connexion (connection refuse)

Les raisons habituelles sont les suivantes :
1. Le client se connecte sur le mauvais port.
2. Le client se connecte sur un domaine ou une adresse IP incorrecte.
3. Si le client utilise un nom de domaine pour se connecter, ce dernier pourrait pointer vers une adresse IP de serveur incorrecte.
4. Le serveur utilise un proxy tel que CDN pour accélérer la connexion, ce qui peut entraîner une incohérence entre l'adresse IP réelle de la connexion et l'adresse IP attendue.
5. Le serveur n'est pas démarré ou le port n'est pas écouté.
6. Un logiciel de proxy réseau est utilisé.
7. L'adresse IP écoutée par le serveur et l'adresse d'accès ne sont pas dans la même plage d'adresses. Par exemple, si le serveur écoute sur 127.0.0.1, le client ne pourra se connecter qu'en utilisant 127.0.0.1, mais pas avec une adresse IP de réseau local ou une adresse IP externe. Il est recommandé de définir l'adresse d'écoute sur 0.0.0.0, de sorte que la connexion puisse se faire à partir de l'ordinateur local, du réseau local et du réseau externe.

## Dépassement du délai de connexion (connection timeout)

Les raisons habituelles sont les suivantes :
1. Le pare-feu du serveur bloque la connexion. Essayez de désactiver temporairement le pare-feu.
2. Si vous utilisez un serveur cloud, le groupe de sécurité peut également bloquer l'établissement de la connexion. Vous devrez ouvrir les ports correspondants dans le tableau de bord de gestion.
3. Si vous utilisez des panneaux de contrôle tels que BaoTa, vous devrez ouvrir les ports correspondants à partir de ce panneau de contrôle.
4. Le serveur n'existe pas ou n'a pas été démarré.
5. Si le client utilise un nom de domaine pour se connecter, ce dernier pourrait pointer vers une adresse IP de serveur incorrecte.
6. L'adresse IP à laquelle le client accède est une adresse IP interne du serveur et le client et le serveur ne sont pas dans le même réseau local.

## Impossible d'attribuer l'adresse demandée (cannot assign requested address)

**Lors de l'utilisation en tant que client**, chaque connexion initiée nécessite l'utilisation d'un port local temporaire. Par défaut, un serveur dispose généralement de 20 000 à 30 000 ports temporaires disponibles. Si le nombre de connexions initiées vers un serveur particulier dépasse cette valeur, il ne sera pas possible d'attribuer un port disponible, ce qui entraînera cette erreur.
Vous pouvez augmenter le nombre de ports temporaires locaux en modifiant le paramètre du noyau `/etc/sysctl.conf` `net.ipv4.ip_local_port_range`, par exemple en le définissant sur `10000 65535` (ce qui ajoute 55 535 ports locaux), puis en exécutant `sysctl -p` pour appliquer les changements.
De plus, lorsque la connexion est interrompue, elle reste dans l'état TIME_WAIT, ce qui signifie qu'elle occupe toujours le port local correspondant pendant un certain temps. Ainsi, une grande quantité de connexions courtes (plus de 20 000 à 30 000) initiées sur une courte période générera également une erreur `Cannot assign requested address`. Dans ce cas, vous pouvez résoudre ce problème en configurant le noyau pour libérer rapidement les connexions TIME_WAIT, comme décrit dans la [optimisation du noyau](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html).

> **Remarque**
> La limitation du nombre de ports locaux s'applique uniquement au client. Le serveur n'est pas limité par le nombre de ports locaux et peut maintenir un nombre de connexions indéfini tant que les ressources le permettent.

## Autres erreurs
Si l'erreur rencontrée n'est ni ```refus de connexion``` ni ```dépassement du délai de connexion```, elle est généralement due aux raisons suivantes :

**1. Le protocole de communication utilisé par le client et le serveur n'est pas le même.**
Par exemple, si le serveur utilise le protocole de communication HTTP, le client ne pourra pas se connecter en utilisant le protocole de communication WebSocket. Si le client utilise le protocole WebSocket pour se connecter, alors le serveur doit également utiliser le protocole WebSocket. De même, si le serveur est un service HTTP, le client doit utiliser le protocole HTTP pour la connexion.

Le principe ici est similaire au fait que pour communiquer avec un Britannique, il faut utiliser l'anglais. Pour communiquer avec un Japonais, il faut utiliser le japonais. Ici, le langage est similaire au protocole de communication, les deux parties (client et serveur) doivent utiliser le même langage pour communiquer, sinon la communication est impossible.

**Des erreurs courantes causées par des protocoles de communication incompatibles sont :**

> La connexion WebSocket à 'ws://xxx.com:xx/' a échoué : Erreur lors de l'ouverture de WebSocket : Code de réponse inattendu : xxx

> La connexion WebSocket à 'ws://xxx.com:xx/' a échoué : Erreur lors de l'ouverture de WebSocket : net::ERR_INVALID_HTTP_RESPONSE

**Solutions :**
En examinant ces deux erreurs, nous voyons que le client utilise une connexion WebSocket avec le protocole WebSocket. Le serveur doit donc également utiliser le protocole WebSocket pour que la communication puisse être établie. Par exemple, le code d'écoute du serveur doit spécifier le protocole WebSocket pour que la communication puisse être établie, comme indiqué ci-dessous :

Pour GatewayWorker, une partie du code d'écoute ressemblerait à ceci :
```php
// Protocole WebSocket, afin que le client puisse se connecter avec ws://... xxxx étant le port, sans besoin de modification
$gateway = new Gateway('websocket://0.0.0.0:xxxx');
```
Pour Workerman, ce serait :
```php
// Protocole WebSocket, afin que le client puisse se connecter avec ws://... xxxx étant le port, sans besoin de modification
$worker = new Worker('websocket://0.0.0.0:xxxx');
```
