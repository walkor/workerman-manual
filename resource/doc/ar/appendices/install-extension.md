# تثبيت الامتدادات
## ملاحظة
في مقابل وضع تشغيل Apache+PHP أو Nginx+PHP، يقوم WorkerMan بتشغيله بناء على سطر الأوامر [PHP CLI](http://php.net/manual/zh/features.commandline.php)، مع استخدام برنامج PHP قابل للتنفيذ مختلف وملف php.ini قد يكون مختلفا أيضا. لذا، عندما ترى الاضافات المثبتة باستخدام ```phpinfo()``` على الصفحة، فلا يعني ذلك أن أمتداد الPHP CLI أيضا مثبت.

## كيف تحدد أي امتدادات PHP CLI مثبتة
تقوم بتشغيل```php -m``` لعرض الامتدادات التي تم تثبيتها بالفعل في سطر الأوامر PHP CLI، والنتيجة تكون مماثلة للتالي:
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## كيف تحدد موقع ملف php.ini لـ PHP CLI
عندما نقوم بتثبيت امتدادات، قد نحتاج إلى تكوين ملف php.ini يدويا لإضافة الاضافات، لذا يجب التأكد من موقع ملف php.ini لـ PHP CLI باستخدام```php --ini``، والنتيجة تكون مماثلة للتالي (قد تكون النتائج مختلفة بين الأنظمة):
```shell
~# php --ini
Configuration File (php.ini) Path: /etc/php8/cli
Loaded Configuration File:         /etc/php8/cli/php.ini
Scan for additional .ini files in: /etc/php8/cli/conf.d
Additional .ini files parsed:      /etc/php8/cli/conf.d/apc.ini,
/etc/php8/cli/conf.d/pdo.ini,
/etc/php8/cli/conf.d/pdo_mysql.ini
...
```

# تثبيت الامتدادات لـ PHP CLI (تثبيت امتداد memcached كمثال)
## الطريقة الأولى: استخدام أوامر apt أو yum
إذا كان PHP مثبتا عبر أوامر apt أو yum، يمكن تثبيت الامتدادات باستخدام apt أو yum

**طريقة تثبيت الامتدادات في debian/ubuntu وأنظمة مشابهة باستخدام apt (يجب على غير المستخدمين الجذر إضافة sudo)**
1. استخدام ```apt-cache search``` للبحث عن حزم الامتداد
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2. استخدام ```apt-get install``` لتثبيت حزم الامتداد
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**طريقة تثبيت الامتدادات في centos وأنظمة مشابهة باستخدام yum**
1. استخدام ```yum search``` للبحث عن حزم الامتداد
```shell
~# yum search memcached php
php-pecl-memcached - memcached module for php5
```
2. استخدام ```yum install``` لتثبيت حزم الامتداد
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```
**ملاحظة:**
تثبيت الامتدادات باستخدام apt أو yum سيقوم تلقائيا بتكوين ملف php.ini، وبعد التثبيت يمكن استخدامها مباشرة، ولكن العيب في ذلك هو أن بعض الامتدادات قد لا يكون لها حزم تثبيت متاحة في apt أو yum.

## الطريقة الثانية: استخدام pecl للتثبيت
استخدام أمر ```pecl install``` لتثبيت الامتدادات

1. تثبيت ```pecl install```
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. تكوين ملف php.ini

من خلال تشغيل ```php --ini``` للبحث عن موقع ملف php.ini، ثم قم بإضافة```extension=memcached.so``` في الملف.

## الطريقة الثالثة: تثبيت مُدمج للمصدر
1. استخدام ```php -v``` لعرض إصدار PHP CLI الحالي
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. قم بتنزيل مصدر PHP وفقًا للإصدار

صفحة تحميل الإصدارات التاريخية لـ PHP: https://php.net/releases/

3. فك الضغط عن ملف الضغط
مثلاً في حالة تحميل ملف "php-5.3.29.tar.gz" 
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. انتقل إلى دليل ext/pcntl في المصدر
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. قم بتشغيل الأمر ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. قم بتشغيل أمر ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. قم بتشغيل أمر ```make```
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. قم بتشغيل أمر ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. قم بتكوين ملف ini

من خلال تشغيل ```php --ini``` للبحث عن موقع ملف php.ini، ثم أضف ```extension=pcntl.so``` في الملف.

**ملاحظة:**
يتم استخدام هذه الطريقة عادة لتثبيت امتدادات PHP المضمنة، مثل امتداد posix وامتداد pcntl. بالإضافة إلى استخدام phpize لتجميع امتداد معين، يمكن أيضًا إعادة تجميع PHP بالكامل وإضافة الامتدادات في وقت التجميع، على سبيل المثال يمكن تشغيل الأوامر التالية في دليل المصدر:
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## الطريقة الرابعة: تثبيت باستخدام phpize
إذا كنت ترغب في تثبيت امتداد غير موجود في دليل ext في مصدر PHP، فيجب البحث عن هذا الامتداد وتنزيله من https://pecl.php.net

مثال: تثبيت امتداد libevent (افترض أن النظام قد قام بتثبيت مكتبة libevent-dev)

1. تنزيل ملف الضغط للامتداد libevent (في أي مجلد في النظام الحالي)
```shell
~# wget https://pecl.php.net/get/libevent-0.1.0.tgz
--2015-05-26 21:43:40--  https://pecl.php.net/get/libevent-0.1.0.tgz
Resolving pecl.php.net... 104.236.228.160
Connecting to pecl.php.net|104.236.228.160|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9806 (9.6K) [application/octet-stream]
Saving to: “libevent-0.1.0.tgz”

100%[=======================================================>] 9,806       41.4K/s   in 0.2s
```

2. فك الضغط عن ملف الضغط للامتداد
``shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3. انتقل إلى دليل المصدر
```shell
~# cd libevent-0.1.0/
```

4. قم بتشغيل أمر ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

5. قم بتشغيل أمر ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```

6. قم بتشغيل أمر ```make```
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7. قم بتشغيل أمر ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```

8. قم بتكوين ملف ini

من خلال تشغيل ```php --ini``` للبحث عن موقع ملف php.ini، ثم أضف ```extension=libevent.so``` في الملف.
