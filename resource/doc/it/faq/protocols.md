# Protocolli supportati da WorkerMan

WorkerMan supporta vari protocolli a livello di interfaccia, purché rispettino l'interfaccia ```ConnectionInterface``` (vedere la sezione Protocolli personalizzati). 

Per comodità degli sviluppatori, WorkerMan fornisce supporto per i protocolli HTTP, WebSocket e anche per un protocollo di testo molto semplice, oltre a un protocollo di frame utilizzabile per la trasmissione binaria. Gli sviluppatori possono utilizzare direttamente questi protocolli senza dover fare ulteriori sviluppi. Se nessuno di questi protocolli soddisfa le esigenze, gli sviluppatori possono implementare il proprio protocollo seguendo la sezione Protocolli personalizzati.

Gli sviluppatori possono anche basarsi direttamente su protocolli TCP o UDP.

Esempi di utilizzo dei protocolli:
```php
// Protocollo HTTP
$worker1 = new Worker('http://0.0.0.0:1221');
// Protocollo WebSocket
$worker2 = new Worker('websocket://0.0.0.0:1222');
// Protocollo di testo (protocollo telnet)
$worker3 = new Worker('text://0.0.0.0:1223');
// Protocollo di frame (utilizzabile per la trasmissione binaria)
$worker3 = new Worker('frame://0.0.0.0:1223');
// Basato direttamente su trasmissione TCP
$worker4 = new Worker('tcp://0.0.0.0:1224');
// Basato direttamente su trasmissione UDP
$worker5 = new Worker('udp://0.0.0.0:1225');
```
