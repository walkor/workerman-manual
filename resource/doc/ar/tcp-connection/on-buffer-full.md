# onBufferFull
## الوصف:
```php
callback Connection::$onBufferFull
```

هذا الدالة يؤدي نفس الوظيفة كدالة `Worker::$onBufferFull` مع اختلاف في أنها تعمل فقط على الاتصال الحالي، أي يمكن تعيين دالة `onBufferFull` لاتصال معين منفصلة.
