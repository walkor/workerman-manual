تعطيل فحص الوظائف

استخدم هذا النص للتحقق مما إذا كانت هناك وظائف معطلة. قم بتشغيل هذا الأمر في سطر الأوامر `curl -Ss https://www.workerman.net/check | php` 

إذا كان هناك تنبيه يشير إلى `Function اسم_الوظيفة may be disabled. Please check disable_functions in php.ini` يعني أن الوظائف التي يعتمد عليها workerman قد تم تعطيلها، ويجب رفع التعطيل عنها في ملف php.ini لتمكين استخدام workerman بشكل صحيح.
لتعطيل التعطيل، يمكن اتباع أحد الطرق التالية.

## الطريقة الأولى: حل السكريبت

قم بتنفيذ السكريبت `curl -Ss https://www.workerman.net/fix | php` لرفع التعطيل

## الطريقة الثانية: الرفع يدويًا

**الخطوات التالية:**

1. تشغيل الأمر `php --ini` للعثور على موقع ملف php.ini الذي يستخدمه php cli.

2. افتح ملف php.ini وابحث عن البند `disable_functions` وقم برفع التعطيل عن الوظيفة المقابلة.

**الوظائف المعتمدة**
لاستخدام workerman، يجب رفع التعطيل عن الوظائف التالية:
```stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
