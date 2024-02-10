# onBufferDrain
## Descrição:
```php
callback Connection::$onBufferDrain
```

Funciona da mesma forma que o retorno de chamada [Worker::$onBufferDrain](../worker/on-buffer-drain.md), a diferença é que funciona apenas para a conexão atual, ou seja, você pode definir separadamente um retorno de chamada onBufferDrain para uma conexão específica.
