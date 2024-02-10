# onBufferDrain
## Descripción:
```php
callback Connection::$onBufferDrain
```

Tiene el mismo efecto que la devolución de llamada [Worker::$onBufferDrain](../worker/on-buffer-drain.md), con la diferencia de que solo afecta a la conexión actual, es decir, se puede configurar de forma individual la devolución de llamada onBufferDrain para una conexión específica.
