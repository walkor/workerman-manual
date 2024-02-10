# onError
## 說明:
```php
callback Connection::$onError
```

此函式與[Worker::$onError](../worker/on-error.md)回調功能相同，不同之處在於它僅針對當前連線起作用，也就是可以單獨設置某個連線的onError回調。
