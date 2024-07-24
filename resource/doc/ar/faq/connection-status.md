تشغيل ```php start.php status``` يمكنك رؤية حالة تشغيل ملقم Workerman الحالية. حقل ```connections``` يعلمك عدد اتصالات TCP الحالية لكل عملية. يجب ملاحظة أن هذا الحقل يشمل ليس فقط عدد اتصالات TCP للعملاء، بل يشمل أيضًا عدد اتصالات TCP للتواصل الداخلي في Workerman. على سبيل المثال، في نموذج Gateway/Worker في Workerman، يكون عدد اتصالات عملاء كل عملية Gateway الحالية هو قيمة حقل ```connections``` ناقص عدد عمليات Worker.