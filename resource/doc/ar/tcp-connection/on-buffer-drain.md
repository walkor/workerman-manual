# onBufferDrain
## الوصف:
```php
callback Connection::$onBufferDrain
```

يؤدي نفس الدور المُؤدى بواسطة الاستدعاء [Worker::$onBufferDrain](../worker/on-buffer-drain.md) ، والاختلاف هو أنه يعمل فقط على الاتصال الحالي، أي أنه يمكن تعيين استدعاء onBufferDrain لاتصال معين بشكل منفصل.
