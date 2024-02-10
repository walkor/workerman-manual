# onBufferDrain
## Описание:
```php
callback Connection::$onBufferDrain
```

Действует так же, как и обратный вызов [Worker::$onBufferDrain](../worker/on-buffer-drain.md), но отличие заключается в том, что он действует только для текущего соединения, то есть можно отдельно настроить обратный вызов onBufferDrain для определенного соединения.
