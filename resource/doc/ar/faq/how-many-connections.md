Workerman يدعم عدد من التطابق؟

مصطلح "التطابق" غير واضح جدًا هنا. هناك نوعان من المؤشرات الكمية يمكن استخدامها للتعبير عن "عدد التطابق"، وهما "عدد اتصالات متزامنة" و "عدد طلبات متزامنة".

"عدد الاتصالات المتزامنة" يشير إلى عدد اتصالات TCP التي يحافظ عليها الخادم في لحظة معينة، بصرف النظر عن ما إذا كانت هناك بيانات تتبادل عبر هذه الاتصالات. على سبيل المثال، قد يحتفظ خادم الرسائل بمئات الآلاف من اتصالات الأجهزة، ولكن نظرًا لندرة تبادل البيانات عبر هذه الاتصالات، فقد يكونتحميل الخادم منخفضًا تقريبًا، ويمكنه استمرار استقبال المزيد من الاتصالات طالما أن الذاكرة كافية.

"عدد الطلبات المتزامنة" عادة يتم قياسه بواسطة QPS (عدد الطلبات التي يقوم الخادم بمعالجتها لكل ثانية)، دون النظر بشكل كبير إلى عدد اتصالات TCP على الخادم في الوقت الحالي. على سبيل المثال، إذا كان لدى الخادم فقط 10 اتصالات عميلية، وكان لكل عميل 10،000 طلب في الثانية، فيجب أن يكون الخادم قادرًا على دعم 100,000 طلب في الثانية كحد أدنى (QPS). ويفترض أن 100,000 طلب في الثانية هو الحد الأقصى لهذا الخادم. إذا كان كل عميل يرسل طلبًا واحدًا في الثانية إلى الخادم، فإن الخادم يمكن أن يدعم 100,000 عميل.

"عدد الاتصالات المتزامنة" يعتمد على ذاكرة الخادم، وعادةً ما يمكن لخادم Workerman بذاكرة 24 جيجابايت دعم حوالي 120,000 اتصال متزامن.

"عدد الطلبات المتزامنة" يعتمد على قدرة معالج الخادم، ويمكن لخادم Workerman بـ 24 نواة دعم حوالي 450,000 طلب في الثانية (QPS) كحد أقصى، وقيمه الفعلية تختلف تبعًا لتعقيد العمل وجودة الشفرة.

## ملاحظة

في حالات الأحمال العالية، من الضروري تثبيت الامتداد event، يرجى الرجوع إلى فصل تثبيت وتكوين. كما يتطلب تحسين نواة Linux، خاصة قيود عدد الملفات المفتوحة للعمليات، يرجى الرجوع إلى فصل تحسين نواة المرفقات.

## بيانات الاختبار

> من مؤسسة اختبار الضغط التقني المعترف بها من الجولة 20 للاختبار
https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf

**تكوين الخادم:**
إجمالي النوى 14، إجمالي الخيوط 28، 32 جيجابايت من الذاكرة، مفصولة من Cisco 10-جيجابت إيثرنت سويتش
**المنطق التجاري:**
عمليات قاعدة بيانات بوستجريس، PHP8+jit
QPS هو 390,000+ ![screenshot_1636522357217](../images/screenshot_1636522357217.png)
