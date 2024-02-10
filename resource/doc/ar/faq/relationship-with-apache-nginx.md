# العلاقة بين Apache و Nginx

**السؤال:**

ما هي علاقة Workerman مع Apache/nginx/php-fpm؟ هل يحدث تضارب بين Workerman و Apache/nginx/php-fpm؟

**الإجابة:**

لا توجد أي علاقة بين Workerman و Apache/nginx/php-fpm، وتشغيل Workerman لا يعتمد على Apache/nginx/php-fpm. إنهما يعملان كحاويات مستقلة، ولا يتداخلان أو يتصارعان (بشرط عدم الاستماع على نفس المنفذ).

Workerman هو إطار عام لخوادم المقابس، يدعم الاتصالات الدائمة، ويدعم مجموعة متنوعة من البروتوكولات مثل HTTP وWebSocket وبروتوكولات مخصصة. أما Apache/nginx/php-fpm فعمومًا يستخدم فقط لتطوير مشاريع الويب التي تستخدم بروتوكول HTTP.

إذا كانت الخادم قد نصب بالفعل Apache/nginx/php-fpm، فإن تثبيت Workerman لن يؤثر على تشغيلها.
