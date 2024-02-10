# onBufferDrain
## 설명:
```php
callback Connection::$onBufferDrain
```

이 콜백은 [Worker::$onBufferDrain](../worker/on-buffer-drain.md) 콜백과 동일한 기능을 합니다. 다만 현재 연결에만 적용되는 것이 차이점으로, 특정 연결의 onBufferDrain 콜백을 개별적으로 설정할 수 있습니다.
