# onError
## الوصف:
```php
callback Connection::$onError
```

يعمل بنفس طريقة عمل رد الاستدعاء [Worker::$onError](../worker/on-error.md)، الاختلاف هو أنه يعمل فقط على الاتصال الحالي، وبمعنى آخر يمكن تعيين رد فعل onError لاتصال معين بشكل منفصل
