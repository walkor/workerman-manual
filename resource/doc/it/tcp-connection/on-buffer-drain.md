# onBufferDrain
## Descrizione:
```php
callback Connection::$onBufferDrain
```
Ha la stessa funzione del callback [Worker::$onBufferDrain](../worker/on-buffer-drain.md), ma con la differenza che agisce solo sulla connessione corrente, cioè è possibile impostare separatamente il callback onBufferDrain per una determinata connessione.
