## كيفية تشغيل Workerman تلقائيًا عند تشغيل نظام Linux

افتح ملف /etc/rc.local وأضف الكود التالي مثلًا قبل ```exit 0```

```bash
ulimit -HSn 102400
/usr/bin/env php /مسار/القرص/start.php start -d

exit 0
```
