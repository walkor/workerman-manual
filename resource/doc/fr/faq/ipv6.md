# IPv6

Question: Comment faire pour que les clients puissent accéder à l'adresse IPv4 et à l'adresse IPv6?

Réponse: Lors de l'initialisation du conteneur, il suffit de spécifier l'adresse d'écoute comme ```[::]```.

Par exemple
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```
