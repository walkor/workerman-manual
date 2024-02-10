# onError
## 說明:
```php
回調函數 Connection::$onError
```

其作用與[Worker::$onError](../worker/on-error.md)回調相同，唯一不同之處在於僅針對當前連線有效，也就是可以單獨設置某個連線的 onError 回調。
