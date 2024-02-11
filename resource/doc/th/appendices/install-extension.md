# การติดตั้งส่วนขยาย
## ข้อควรระวัง
การทำงานของ Workerman นั้นต่างจากโหมดการทำงานของ Apache+PHP หรือ Nginx+PHP โดย Workerman จะใช้ PHP CLI ในการทำงานซึ่งเป็นโปรแกรม PHP ที่ใช้งานผ่าน Command line ดังนั้น PHP CLI อาจจะใช้โปรแกรม PHP และ PHP.ini ที่แตกต่างกันไป จึงทำให้ การใช้คำสั่ง ```phpinfo()``` ในหน้าเว็บเพจเพื่อดูว่าได้ทำการติดตั้งส่วนขยายไปแล้ว อาจจะไม่แสดงค่าการติดตั้งของ PHP CLI ได้อย่างถูกต้อง

## การตรวจสอบส่วนขยายที่ถูกติดตั้งใน PHP CLI
การทำงาน ```php -m``` จะแสดงรายการของส่วนขยายที่ได้ถูกติดตั้งไว้ใน PHP CLI ที่ได้แสดงออกมาเช่นนี้
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```
## การตรวจสอบส่วนการตั้งค่า php.ini ของ PHP CLI
เพื่อระบุตำแหน่งของไฟล์ php.ini สามารถทำได้โดยการรันคำสั่ง ```php --ini``` เพื่อค้นหาตำแหน่งของไฟล์ php.ini ของ PHP CLI ที่แสดงผลลัพธ์ออกมาดังต่อไปนี้ (ผลลัพธ์ที่แสดงออกมาอาจมีความแตกต่างกันไปตามระบบปฏิบัติการ)
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
# การติดตั้งส่วนขยายให้กับ PHP CLI (ตัวอย่างการติดตั้งส่วนขยาย memcached)
## วิธีที่ 1: ใช้คำสั่ง apt หรือ yum ในการติดตั้ง
หาก PHP ได้ถูกติดตั้งผ่าน apt หรือ yum จะสามารถใช้คำสั่ง apt หรือ yum เพื่อทำการติดตั้งส่วนขยายได้ดังนี้

**วิธีการติดตั้งส่วนขยายในระบบ debian/ubuntu และอื่น ๆ (ผู้ใช้ที่ไม่ใช่ root ต้องใช้คำสั่ง sudo)**
1. ค้นหาแพ็คเกจส่วนขยายโดยใช้ ```apt-cache search```
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2. ใช้คำสั่ง ```apt-get install``` เพื่อทำการติดตั้งแพ็คเกจส่วนขยาย
```shell
~# apt-get install -y php5-memcached
```
**วิธีการติดตั้งส่วนขยายในระบบ centos และอื่น ๆ**
1. ค้นหาแพ็คเกจส่วนขยายโดยใช้คำสั่ง ```yum search```
```shell
~# yum search memcached php
php-pecl-memcached - memcached module for php5
```
2. ใช้คำสั่ง ```yum install``` เพื่อทำการติดตั้งแพ็คเกจส่วนขยาย
```shell
~# yum install -y php-pecl-memcached
```
**หมายเหตุ：**
การติดตั้งส่วนขยาย PHP ด้วย apt หรือ yum จะทำการตั้งค่าไฟล์ php.ini โดยอัตโนมัติ หลังจากการติดตั้งเสร็จสมชื่อ สามารถใช้งานได้ทันที แต่บางครั้งอาจมีข้อเสียหาที่บางส่วนจะไม่มีแพ็คเกจติดตั้งส่วนขยายที่ต้องใช้

## วิธีที่ 2: ใช้ pecl ในการติดตั้ง
การใช้คำสั่ง ```pecl install``` ในการติดตั้งส่วนขยายแบบต่าง ๆ ดังนี้

1. ทำการติดตั้งด้วยคำสั่ง ```pecl install```
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. ปรับแต่งไฟล์ php.ini

โดยการทำงาน ```php --ini``` เพื่อค้นหาตำแหน่งของไฟล์ php.ini และจากนั้นเพิ่ม ```extension=memcached.so``` ลงในไฟล์ดังกล่าว

## วิธีที่ 3: การติดตั้งจาก source code (การติดตั้งส่วนขยาย pcntl เป็นตัวอย่าง)
1. ใช้คำสั่ง ```php -v``` เพื่อดูเวอร์ชันของ PHP CLI ปัจจุบัน
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. ดาวน์โหลด source code ของ PHP ตามเวอร์ชันที่ได้ทำการติดตั้ง

หากต้องการดาวน์โหลดเวอร์ชันเก่าของ PHP สามารถทำได้ที่ https://php.net/releases/

3. แตก source code จากไฟล์ที่ดาวน์โหลดมา

เช่น หากได้ดาวน์โหลดไฟล์ชื่อ ```php-5.3.29.tar.gz```
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. เข้าไปในไดเรกทอรี ext/pcntl ที่อยู่ใน source code
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. รันคำสั่ง ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. รันคำสั่ง ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. รันคำสั่ง ```make```
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. รันคำสั่ง ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. ปรับแต่งไฟล์ ini

โดยการทำงาน ```php --ini``` เพื่อค้นหาตำแหน่งของไฟล์ php.ini และจากนั้นเพิ่ม ```extension=pcntl.so``` ลงในไฟล์ดังกล่าว

**หมายเหตุ :**
วิธีนี้จะใช้สำหรับการติดตั้งส่วนขยายที่เป็นของ PHP อย่างเช่น ส่วนขยาย posix และ pcntl นอกจากนี้ยังสามารถใช้ phpize เพื่อคอมไพล์ส่วนขยายบางรายการ และยังสามารถทำการคอมไพล์ PHP ใหม่อีกต่อการคอมไพล์โดยกำหนดตัวเลือกส่วนขยายเพิ่มเติม เช่น การรันคำสั่งต่อไปนี้ที่ดำเนินการในไดเร็กทอรีหลักของ Source code

```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```
## ขั้นตอนที่สี่: การติดตั้งโดยใช้ phpize
หากต้องการติดตั้งส่วนขยายที่ไม่มีในไดเรกทอรี ext ของโค้ดภายในของ PHP คุณจำเป็นต้องดาวน์โหลดไฟล์ส่วนขยายจาก https://pecl.php.net ดังนี้

### ตัวอย่างขั้นตอนการติดตั้งส่วนขยาย libevent (สมมติว่าระบบได้ติดตั้งไลบรารี libevent-dev ไว้แล้ว)

1. ดาวน์โหลดไฟล์บีบอัดของส่วนขยาย libevent (สามารถดาวน์โหลดไปไว้ในไดเรกทอรีใดก็ได้ในระบบปัจจุบัน)
```shell
~# wget https://pecl.php.net/get/libevent-0.1.0.tgz
--2015-05-26 21:43:40--  https://pecl.php.net/get/libevent-0.1.0.tgz
กำลังแก้ไขโดเมน pecl.php.net... 104.236.228.160
กำลังเชื่อมต่อกับ pecl.php.net|104.236.228.160|:80... เชื่อมต่อแล้ว
กำลังร้องขอ HTTP แล้วรอการตอบรับ... 200 OK
ยาว: 9806 (9.6K) [application/octet-stream]
กำลังบันทึกเป็น: “libevent-0.1.0.tgz”

100%[=======================================================>] 9,806       41.4K/s   in 0.2s
```

2. แตกไฟล์จากไฟล์บีบอัดของส่วนขยาย
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3. เข้าไปในไดเรกทอรีของโค้ด
```shell
~# cd libevent-0.1.0/
```

4. ทำการรันคำสั่ง ``` phpize ```
```shell
~# phpize
กำลังกำหนดค่าสำหรับ:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

5. ทำการรันคำสั่ง ``` configure ```
```shell
~# ./configure
กำลังตรวจสอบ grep ที่จะจัดการตัวอักษรยาวและ -e... /bin/grep
กำลังตรวจสอบ egrep... /bin/grep -E
กำลังตรวจสอบ sed ที่ไม่ต้องตัดผลลัพธ์... /bin/sed
กำลังตรวจสอบ cc... cc
กำลังตรวจสอบว่าคอมไพเลอร์ C ทำงาน... ใช่
...
```

6. ทำการรันคำสั่ง ``` make ```
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7. ทำการรันคำสั่ง ``` make install ```
```shell
~# make install
กำลังทำการติดตั้งส่วนขยายที่ใช้ร่วม:     /usr/lib/php5/20090626/
```

8. ตั้งค่าไฟล์ ini

โดยการใช้คำสั่ง ```php --ini``` ให้ค้นหาตำแหน่งของไฟล์ php.ini และเพิ่ม ```extension=libevent.so``` ลงไปในไฟล์นั้น
