# onBufferDrain
## Description:
```php
callback Connection::$onBufferDrain
```

This has the same effect as the [Worker::$onBufferDrain](../worker/on-buffer-drain.md) callback, the difference is that it only applies to the current connection, meaning that you can set the onBufferDrain callback for a specific connection individually.
