# Quels protocoles sont supportés par WorkerMan

WorkerMan prend en charge divers protocoles au niveau de l'interface, tant qu'ils sont conformes à l'interface ```ConnectionInterface``` (voir la section sur la personnalisation des protocoles).

Pour faciliter le développement, WorkerMan fournit le support des protocoles HTTP, WebSocket, ainsi que des protocoles de texte très simples, pouvant être utilisés pour des transmissions binaires. Les développeurs peuvent utiliser directement ces protocoles, sans avoir besoin de développer à nouveau. Si aucun de ces protocoles ne répond aux besoins, les développeurs peuvent suivre la section sur la personnalisation des protocoles pour implémenter leur propre protocole.

Les développeurs peuvent également utiliser directement les protocoles TCP ou UDP.

Exemples d'utilisation des protocoles
```php
// protocole HTTP
$worker1 = new Worker('http://0.0.0.0:1221');
// protocole WebSocket
$worker2 = new Worker('websocket://0.0.0.0:1222');
// protocole de texte (protocole telnet)
$worker3 = new Worker('text://0.0.0.0:1223');
// protocole de frame (utilisable pour la transmission binaire)
$worker3 = new Worker('frame://0.0.0.0:1223');
// transmission directe basée sur TCP
$worker4 = new Worker('tcp://0.0.0.0:1224');
// transmission directe basée sur UDP
$worker5 = new Worker('udp://0.0.0.0:1225');
```
