# onBufferFull
## Description:
```php
callback Connection::$onBufferFull
```

This function is the same as the [Worker::$onBufferFull](../worker/on-buffer-full.md) callback, but it only applies to the current connection. In other words, it can be used to set the onBufferFull callback for a specific connection individually.
