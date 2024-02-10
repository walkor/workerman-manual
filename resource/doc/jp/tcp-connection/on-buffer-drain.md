# onBufferDrain
## 説明:
```php
callback Connection::$onBufferDrain
```

[Worker::$onBufferDrain](../worker/on-buffer-drain.md) コールバックと同様の機能を持ちますが、異なるのは現在の接続にのみ適用される点です。つまり、特定の接続のonBufferDrainコールバックを個別に設定することができます。
