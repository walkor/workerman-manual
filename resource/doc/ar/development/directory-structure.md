# هيكل المجلدات

```
Workerman                      // كود نواة Workerman
    ├── Connection                 // ذات الصلة بالاتصالات الداخل
    │   ├── ConnectionInterface.php// واجهة الاتصال بالمقبس
    │   ├── TcpConnection.php      // فئة الاتصال بالـ Tcp
    │   ├── AsyncTcpConnection.php // فئة الاتصال بالـ Tcp اللازم
    │   └── UdpConnection.php      // فئة الاتصال بالـ Udp
    ├── Events                     // مكتبة أحداث الشبكة
    │   ├── EventInterface.php     // واجهة مكتبة أحداث الشبكة
    │   ├── Event.php              // مكتبة أحداث الشبكة Libevent
    │   ├── Ev.php                 // مكتبة أحداث الشبكة Libev
    │   ├── Swoole.php             // مكتبة أحداث الشبكة Swoole
    │   └── Select.php             // مكتبة أحداث الشبكة Select
    ├── Lib                        // مكتبات شائعة الاستخدام
    │   ├── Constants.php          // تعريف الثوابت
    │   └── Timer.php              // المؤقت
    ├── Protocols                  // ذات الصلة بالبروتوكولات
    │   ├── ProtocolInterface.php  // فئة واجهة البروتوكول
    │   ├── Http                   // ذات الصلة ببروتوكول Http
    │   │   ├── Chunk.php    // فئة chunk الخاصة بـ Http
    │   │   ├── Request.php  // فئة الطلب الخاصة بـ Http
    │   │   ├── Response.php  // فئة الاستجابة الخاصة بـ Http
    │   │   ├── ServerSentEvents.php  // فئة SSE
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // التخزين بالملفات للجلسة
    │   │   │   └── RedisSessionHandler.php // التخزين بالريدس للجلسة
    │   │   ├── Session.php  // فئة الجلسة
    │   │   └── mime.types   // ملف تعيين نوع الوسائط
    │   ├── Http.php               // تنفيذ بروتوكول Http
    │   ├── Text.php               // تنفيذ بروتوكول Text
    │   ├── Frame.php              // تنفيذ بروتوكول Frame
    │   └── Websocket.php          // تنفيذ بروتوكول websocket
    ├── Worker.php                 // Worker
    ├── WebServer.php              // WebServer
    └── Autoloader.php             // تحميل التلقائي للفئات
```
