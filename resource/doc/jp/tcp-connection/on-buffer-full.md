# onBufferFull
## 説明:
```php
callback Connection::$onBufferFull
```

このコールバックは、[Worker::$onBufferFull](../worker/on-buffer-full.md)コールバックと同じ機能を持ちますが、異なるのは現在の接続にのみ対応する点です。つまり、特定の接続のonBufferFullコールバックを個別に設定できます。
