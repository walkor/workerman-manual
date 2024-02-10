# ¿Qué protocolos admite WorkerMan?

WorkerMan admite varios protocolos a nivel de interfaz, siempre que cumplan con la interfaz ```ConnectionInterface``` (consulte la sección de Protocolos de comunicación personalizados).

Para mayor comodidad de los desarrolladores, WorkerMan proporciona soporte para los protocolos HTTP, WebSocket, así como un protocolo de texto muy simple que se puede utilizar para la transmisión binaria. Los desarrolladores pueden usar directamente estos protocolos sin necesidad de realizar más desarrollo. Si ninguno de estos protocolos satisface las necesidades, los desarrolladores pueden implementar su propio protocolo siguiendo la sección de protocolos personalizados.

Los desarrolladores también pueden basarse directamente en los protocolos TCP o UDP.

Ejemplo de uso de protocolos:

```php
// Protocolo HTTP
$worker1 = new Worker('http://0.0.0.0:1221');
// Protocolo WebSocket
$worker2 = new Worker('websocket://0.0.0.0:1222');
// Protocolo de texto (protocolo telnet)
$worker3 = new Worker('text://0.0.0.0:1223');
// Protocolo de trama (utilizado para la transmisión binaria)
$worker3 = new Worker('frame://0.0.0.0:1223');
// Transporte directo a través de TCP
$worker4 = new Worker('tcp://0.0.0.0:1224');
// Transporte directo a través de UDP
$worker5 = new Worker('udp://0.0.0.0:1225');
```
