# onBufferFull
## Описание:
```php
callback Connection::$onBufferFull
```

Этот коллбэк аналогичен коллбэку [Worker::$onBufferFull](../worker/on-buffer-full.md), но работает только для текущего соединения, то есть можно установить коллбэк onBufferFull отдельно для определенного соединения.
