# onBufferFull
## 說明:
```php
回調函數 Connection::$onBufferFull
```

其作用與[Worker::$onBufferFull](../worker/on-buffer-full.md)回調函數相同，不同之處在於它僅對當前連接起作用，即可單獨設置某個連接的onBufferFull回調函數。
