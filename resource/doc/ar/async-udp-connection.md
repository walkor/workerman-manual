# الاتصال بالـ UDP غير المتزامن

**(يتطلب workerman >= 3.0.8)**

يمكن للاتصال بالـ UDP غير المتزامن أن يعمل كعميل UDP للتواصل مع خادم UDP عن بعد.

في الواقع، UDP ليس له اتصال، ولكن لسهولة الاستخدام، يتم الاحتفاظ هنا بقواعد التسمية والواجهة الأساسية مع AsyncTcpConnection.

**ملاحظة: بالنسبة لـ AsyncUdpConnection، فإنها لا تدعم الخصائص أو الأساليب التالية:**
1. لا خاصية connection->id
2. لا خاصية connection->worker
3. لا خاصية connection->transport
4. لا خاصية connection->maxSendBufferSize
5. لا خاصية connection->defaultMaxSendBufferSize
6. لا خاصية connection->maxPackageSize
7. لا استدعاء رد فعل onBufferFull
8. لا استدعاء رد فعل onBufferDrain
9. لا استدعاء رد فعل onError
10. لا واجهة destroy() للاستدعاء
11. لا واجهة pauseRecv() للاستدعاء
12. لا واجهة resumeRecv() للاستدعاء
13. لا واجهة pipe() للاستدعاء
14. لا واجهة reconnect() للاستدعاء

**الخصائص أو الأساليب المدعومة بواسطة AsyncUdpConnection:**
1. تدعم الخاصية connection->protocol
2. تدعم استدعاء رد فعل onMessage
3. تدعم واجهة connect()
4. تدعم واجهة send()
5. تدعم واجهة getRemoteIp()
6. تدعم واجهة getRemotePort()
7. تدعم استدعاء رد فعل onClose.
ملحوظة: نظرًا لأن TCP يعتمد على الاتصال، في الحالات العامة، عند قطع أي طرف اتصال باستخدام close، يمكن لكل طرف تشغيل رد فعل onClose. ومع ذلك، UDP ليس له اتصال، وبالتالي، فإن استدعاء connection->close() يؤدي إلى تنشيط استدعاء onClose المحلي فقط، ولا يمكن تنشيط استدعاء onClose للجانب البعيد.
