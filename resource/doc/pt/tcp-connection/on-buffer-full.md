# onBufferFull
## Descrição:
```php
callback Connection::$onBufferFull
```

Funciona da mesma forma que o retorno de chamada [Worker::$onBufferFull](../worker/on-buffer-full.md), com a diferença de que ele só funciona para a conexão atual, ou seja, pode-se configurar separadamente o retorno de chamada onBufferFull para uma determinada conexão.
