# onBufferFull
## 说明:
```php
callback Connection::$onBufferFull
```

作用与[Worker::$onBufferFull](../worker/on-buffer-full.md)回调相同，区别是只针对当前连接起作用，即可以单独设置某个连接的onBufferFull回调
