# onBufferDrain
## 說明:
```php
callback Connection::$onBufferDrain
```

這個回調函數與[Worker::$onBufferDrain](../worker/on-buffer-drain.md)的作用相同，不同之處在於它僅針對當前連接生效，也就是可以單獨設置某個連接的onBufferDrain回調。
