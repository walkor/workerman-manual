```php
callback Connection::$onError
```

The purpose is the same as the [Worker::$onError](../worker/on-error.md) callback, but it only applies to the current connection. This means it can be used to set the onError callback for a specific connection individually.
