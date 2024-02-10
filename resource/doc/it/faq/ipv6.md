# ipv6

Domanda: Come fare in modo che i client possano accedere tramite un indirizzo IPv4 e anche tramite un indirizzo IPv6?

Risposta: Durante l'inizializzazione del contenitore, basta specificare l'indirizzo di ascolto come ```[::]```.

Ad esempio
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```
