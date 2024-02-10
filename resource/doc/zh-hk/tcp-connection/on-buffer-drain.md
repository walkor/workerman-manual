# onBufferDrain
## 說明:
```php
callback Connection::$onBufferDrain
```

這個回調與[Worker::$onBufferDrain](../worker/on-buffer-drain.md)相同，唯一的區別是它僅對當前連接起作用，也就是可以單獨設置某個連接的onBufferDrain回調。
