# onBufferFull
## 說明:
```php
callback Connection::$onBufferFull
```

與 [Worker::$onBufferFull](../worker/on-buffer-full.md) 回調具有相同功能，不同之處在於它僅針對當前連接有效，也就是可以單獨設置某個連接的 onBufferFull 回調。
