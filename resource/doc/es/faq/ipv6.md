# ipv6

Pregunta: ¿Cómo hacer que los clientes puedan acceder tanto a través de direcciones ipv4 como de direcciones ipv6?

Respuesta: Simplemente use la dirección de escucha ```[::]``` al inicializar el contenedor.

Por ejemplo:
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```
