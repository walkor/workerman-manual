# Préambule

**Workerman, un conteneur d'application PHP à haute performance**

## Qu'est-ce que Workerman ?
Workerman est un conteneur d'application PHP à haute performance, open source, développé en PHP pur.

Workerman n'est pas une réinvention de la roue. Ce n'est pas un cadre MVC, mais un cadre de service plus basique et plus généralisé. Vous pouvez l'utiliser pour développer des proxy TCP, des proxies VPN, des serveurs de jeux, des serveurs de messagerie, des serveurs FTP, voire même développer une version PHP de Redis, une version PHP de base de données, une version PHP de Nginx, une version PHP de PHP-FPM, et bien d'autres. Workerman peut être considéré comme une innovation dans le domaine du PHP, libérant complètement les développeurs de la limitation selon laquelle PHP ne peut être utilisé que pour le web.

En réalité, Workerman ressemble à une version PHP de Nginx, avec comme cœur des processus multiples, Epoll, et des entrées/sorties non bloquantes. Chaque processus Workerman peut maintenir des dizaines de milliers de connexions concurrentes. Étant donné qu'il réside en permanence en mémoire, sans dépendre d'Apache, de Nginx, de PHP-FPM, il offre des performances extrêmement élevées. Il prend en charge TCP, UDP, UNIXSOCKET, les connexions durables, Websocket, HTTP, WSS, HTTPS, ainsi que divers protocoles personnalisés. Il dispose de nombreux composants à haute performance, tels que des minuteries, des clients socket asynchrones, Redis asynchrone, HTTP asynchrone, et bien d'autres.

## Quelques domaines d'application de Workerman
Contrairement aux cadres MVC traditionnels, Workerman peut être utilisé non seulement pour le développement web, mais également dans des domaines plus vastes tels que la messagerie instantanée, l'Internet des objets, les jeux, la gouvernance des services, d'autres serveurs ou middleware, élargissant ainsi considérablement le champ de vision des développeurs PHP. Actuellement, il y a une pénurie de développeurs PHP dans ces domaines. Si vous souhaitez avoir un avantage technologique dans le domaine du PHP, et ne pas vous contenter du CRUD quotidien, ou si vous souhaitez vous orienter vers l'architecture ou devenir un expert technique, Workerman est un cadre très recommandé pour l'apprentissage. Les développeurs sont invités à non seulement maîtriser l'utilisation de Workerman, mais aussi à développer leurs propres projets open source basés sur Workerman afin d'améliorer leurs compétences et accroître leur influence, comme l'[exemple du cadre de scrapping en réseau multi-processus de Beanbun](https://github.com/kiddyuchina/Beanbun), qui a reçu de nombreuses critiques favorables peu de temps après sa mise en ligne.

### Quelques domaines d'application de Workerman incluent :

1. Messagerie instantanée
   Par exemple, chat en direct sur le web, envoi de messages instantanés, mini programmes WeChat, envoi de messages sur des applications mobiles, envoi de messages sur des logiciels PC, etc.
   [[Exemples de la salle de chat workerman-chat](https://www.workerman.net/workerman-chat), [messages Web en direct](https://www.workerman.net/web-sender), [salle de chat Tadpole](https://www.workerman.net/workerman-todpole)]

2. Internet des objets
   Par exemple, la communication avec les imprimantes, les microcontrôleurs, les bracelets intelligents, les appareils domestiques intelligents, les vélos en libre-service, etc.
   [Exemples de cas clients tels que Yilianyun, YiboshiDai etc.]

3. Serveurs de jeux
   Par exemple, les jeux de société, les jeux MMORPG, etc.
   [[Exemple de browserquest-php](https://www.workerman.net/browserquest)]

4. Services HTTP
   Par exemple, développement d'interfaces HTTP à haute performance et de sites web à haute performance. Si vous souhaitez développer des services ou des sites web HTTP, nous recommandons fortement [webman](https://github.com/walkor/webman).

5. Services SOA
   Utiliser Workerman pour encapsuler les différentes unités de fonctionnalités commerciales existantes en tant que services, fournissant ainsi une interface unifiée vers l'extérieur, pour obtenir un système peu couplé, facile à entretenir, hautement disponible, et extensible.
   [[Exemple de workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc), [workerman-thrift](https://github.com/walkor/workerman-thrift)]

6. Autres logiciels serveur
   Par exemple, [GatewayWorker](https://www.workerman.net/doc/gateway-worker), [PHPSocket.IO](https://www.workerman.net/phpsocket_io), [proxy HTTP](https://github.com/walkor/php-http-proxy), [proxy socks5](https://github.com/walkor/php-socks5), [composant de communication distribuée](https://github.com/walkor/Channel), [composant de partage de variables distribuées](https://github.com/walkor/GlobalData), [filet de message](https://github.com/walkor/workerman-queue), serveur DNS, serveur Web, serveur CDN, serveur FTP, etc.

7. Composants
   Tels que [Redis asynchrone](components/workerman-redis.md), [client HTTP asynchrone](components/workerman-http-client.md), [client MQTT IoT](components/workerman-mqtt.md), file d'attente de messages [workerman/redis-queue](components/workerman-redis-queue.md), [workerman/stomp](components/workerman-stomp.md), [workerman/rabbitmq](components/workerman-rabbitmq.md), [composant de surveillance de fichiers](components/file-monitor.md), ainsi que de nombreux autres cadres de composants développés par des tiers, etc.

Il est évident que les cadres MVC traditionnels ont du mal à réaliser les fonctionnalités susmentionnées, c'est donc la raison de la naissance de Workerman.

## Philosophie de Workerman
Simplicité, stabilité, haute performance, distribution.

### **Simplicité**
Le minimalisme fait la beauté. Le noyau de Workerman est très simple, composé de quelques fichiers PHP et ne comprend que quelques interfaces, ce qui le rend très simple à apprendre. Toutes les autres fonctionnalités sont étendues par l'intermédiaire de composants.

Workerman dispose d'une documentation complète, d'un site Web officiel renommé, d'une communauté active, de plusieurs groupes QQ de milliers de membres, de nombreux composants à haute performance, et de nombreux exemples, qui rendent l'utilisation de Workerman plus facile pour les développeurs.

### **Stabilité**
Workerman est open source depuis plusieurs années, et est utilisé à grande échelle par de nombreuses entreprises cotées en bourse, avec une stabilité extrême. Certains services n'ont pas été redémarrés pendant plus de 2 ans et fonctionnent encore à pleine vitesse. Aucun core dump, aucune fuite de mémoire, aucun bug.

### **Haute performance**
En raison de sa résidence en mémoire, de son absence de dépendance à Apache/Nginx/PHP-FPM, de l'absence de surcoût de communication entre conteneurs et PHP, et de l'absence de surcoût d'initialisation et de destruction de toutes les demandes, Workerman offre des performances extrêmement élevées. Par rapport aux cadres MVC traditionnels, les performances sont plusieurs fois supérieures. Sous PHP7, les tests de charge ab ont même montré un QPS supérieur à celui de Nginx seul.

### **Distribution**
Nous ne vivons plus à une époque où l'on peut agir seul. Même si la performance d'un seul serveur est très puissante, il a ses limites. Le déploiement distributeur sur plusieurs serveurs est la voie à suivre. Workerman offre directement une solution de communication distribuée à long terme [GatewayWorker Framework](https://doc2.workerman.net), qui ne nécessite que de simples configurations et le démarrage pour ajouter des serveurs, sans aucun changement dans le code métier, augmentant ainsi la capacité du système de manière exponentielle. Si vous développez des applications à long terme TCP, nous vous recommandons d'utiliser directement [GatewayWorker](https://doc2.workerman.net), qui encapsule Workerman et offre des interfaces plus riches et des capacités de traitement distribuées puissantes pour les applications à long terme.

## Champ d'application du présent manuel
Versions 3.x - 4.x de Workerman

## Utilisateurs de Windows (à lire impérativement)
Workerman prend en charge les systèmes Linux et Windows. La version Windows de Workerman **ne dépend d'aucune extension**, elle nécessite simplement la configuration de la variable d'environnement PHP. Pour connaître les détails de l'installation de la version Windows de Workerman et les points à retenir, consultez le lien [Windows Users Must Read](https://www.workerman.net/windows).

## Client
Le protocole de communication de WorkerMan est ouvert et personnalisable, de ce fait, en théorie, WorkerMan peut communiquer avec des clients utilisant n'importe quel protocole sur n'importe quelle plateforme. Lorsque l'utilisateur développe un client, il peut se baser sur le protocole de communication correspondant pour communiquer avec le serveur.


