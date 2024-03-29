# كيفية التكامل مع الأطر الأخرى
**سؤال:**

كيفية التكامل مع أطر MVC الأخرى (مثل thinkPHP، Yii إلخ)؟

**إجابة:**

![workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

يُقترح **التكامل** مع أطر MVC الأخرى بالطريقة الموضحة في الصورة أعلاه (مثال باستخدام ThinkPHP):

1. تعتبر ThinkPHP و Workerman نظامان مستقلان، يمكن نشرهما بشكل مستقل (يمكن نشرهما على خوادم مختلفة) دون تداخل.
2. يوفر ThinkPHP الصفحات الويب للعرض في المتصفح عبر بروتوكول HTTP.
3. يقوم الـ js المقدم من صفحات ThinkPHP بإنشاء اتصال websocket مع Workerman.
4. بعد الاتصال، يتم إرسال حزمة بيانات إلى Workerman (تحتوي على اسم المستخدم وكلمة المرور أو رمز مميز) للتحقق من أن اتصال websocket ينتمي إلى أي مستخدم.
5. يتم استدعاء واجهة socket في Workerman فقط عندما يحتاج ThinkPHP إلى دفع البيانات إلى المتصفح.
6. يُعالج باقي الطلبات بالطريقة الأصلية لـ ThinkPHP عبر بروتوكول HTTP.

**التلخيص:**

يُعتبر Workerman قناة يمكن من خلالها دفع البيانات إلى المتصفح، ويتم استدعاء واجهة Workerman فقط عندما يتطلب ذلك دفع البيانات إلى المتصفح. يتم إكمال منطق الأعمال بأكمله في ThinkPHP.

للمزيد عن كيفية استدعاء واجهة socket في Workerman من قِبل ThinkPHP لدفع البيانات، يُرجى الرجوع إلى الكتابة عن [كيفية الدفع في مشروع آخر](push-in-other-project.md) في قسم الأسئلة الشائعة.

**دعم ThinkPHP رسميًا لـ Workerman متوفر، راجع [دليل ThinkPHP5](https://www.kancloud.cn/manual/thinkphp5/235128)**
