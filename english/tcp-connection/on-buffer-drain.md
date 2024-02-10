# onBufferDrain
## Description:
```php
callback Connection::$onBufferDrain
```

The function is the same as the [Worker::$onBufferDrain](../worker/on-buffer-drain.md) callback, with the difference that it only applies to the current connection. This means that the onBufferDrain callback can be set individually for a specific connection.
