يمكن استخدام workerman كعميل لتلقي ومعالجة البيانات من الخادم البعيد؟


يمكن استخدام AsyncTcpConnection لإجراء اتصال غير متزامن والسماح لـ workerman بالتفاعل كعميل مع الخادم.

على سبيل المثال، في الأمثلة التالية:
1. [workerman كعميل websocket](as-wss-client.md)
2. [workerman كوكيل mysql](../async-tcp-connection/connect.md)
3. [workerman كعميل http](../async-tcp-connection/construct.md)
4. [workerman كوكيل http](https://github.com/walkor/php-http-proxy)
5. [workerman كوكيل socks5](https://github.com/walkor/php-socks5)
