# คำแนะนำในการติดตั้ง
WorkerMan ซึ่งเป็นจริง ๆ แล้วเป็นแค่ชุดโค้ด PHP หากคุณมี PHP environment ที่ถูกต้องแล้ว คุณสามารถดาวน์โหลดโค้ดต้นฉบับของ WorkerMan หรือตัวอย่างไปทำงานได้ทันที

**การติดตั้งผ่าน Composer:**
```sh
composer require workerman/workerman
```

> **ข้อควรระวัง**
> บางกากเครื่องมีการติดตั้ง composer proxy ที่ไม่ครบ คุณสามารถใช้คำสั่งต่อไปนี้ `composer config -g --unset repos.packagist`  เพื่อลบ proxy

# สำหรับผู้ใช้ Windows (ต้องอ่าน)
ตั้งแต่ workerman เวอร์ชัน 3.5.3 เป็นต้นไป workerman สามารถรองรับทั้งระบบปฏิบัติการ Windows และ Linux ด้วย ผู้ใช้ Windows จำเป็นต้องทำการตั้งค่าตัวแปรสภาพแวดล้อมของ PHP

 ` ===ข้อมูลดังต่อไปนี้เฉพาะสำหรับระบบปฏิบัติการ Linux เท่านั้น ผู้ใช้ Windows โปรดละเว้น=== `

# ตรวจสอบสภาพแวดล้อมระบบปฏิบัติการ Linux
สำหรับระบบปฏิบัติการ Linux คุณสามารถใช้สคริปต์นี้เพื่อทดสอบว่าสภาพแวดล้อม PHP บนเครื่องของคุณพร้อมที่จะรัน WorkerMan หรือไม่
 `curl -Ss https://www.workerman.net/check | php`

หากสคริปต์ด้านบนแสดงข้อความ "ok" ทั้งหมด นั่นหมายความว่า WorkerMan สามารถรันได้โดยตรง คุณสามารถดาวน์โหลดตัวอย่างจาก[เว็บไซต์อย่างเป็นทางการ](https://www.workerman.net/) เพื่อทดสอบใช้งาน

หากมีข้อผิดพลาดสามารถอ้างอิงเอกสารด้านล่างเพื่อทำการติดตังเพิ่มเติม

(หมายเหตุ: สคริปต์ตรวจสอบไม่ได้ทำการตรวจสอบ event extension ถ้าคุณมีการเชื่อมต่อพร้อมโหมดทำงานมากกว่า 1024 ควรติดตัง event extension และ[แก้ไข kernel บนระบบปฏิบัติการ Linux](../appendices/kernel-optimization.md) สามารถอ้างอิงการติดตั้งเพิ่มได้ทางโดยละข้อสั่งด้านล่าง)

# ติดตั้ง extension ที่ขาดหายไปใน PHP environment ที่มีอยู่แล้ว

## ติดตัง extension pcntl และ posix:

**ระบบ centos**
หาก PHP ถูกติดตั้งผ่าน yum คุณสามารถใช้คำสั่งต่อไปนี้เพื่อติดตั้ง extension pcntl และ posix
```shell
yum install php-process
```

หากการติดตั้งไม่สำเร็จ หรือ PHP ไม่ได้ถูกติดตั้งผ่าน yum กรุณาอ้างอิงที่[manual appendices- installation extension](../appendices/install-extension.md) เทเฉพาะเมทอทอดการติดตั้งที่สามเพื่อการสำรอง

**Debian / Ubuntu / MacOS**
กรุณาอ้างอิงที่[manual appendices- installation extension](../appendices/install-extension.md) ส่วนที่สามเพื่อการติดตั้งด้วยโค้ดชุดที่ถูกคอมไพล์ที่สุด

## ติดตั้ง event extension:
เพื่อให้สามารถรองรับจำนวนการเชื่อมต่อที่มากขึ้น คุณจำเป็นจะต้อง ติดตั้ง event extension และ[ปรับแก้ kernel บนระบบปฏิบัติการ Linux](../appendices/kernel-optimization.md) ด้านล่างเป็นช่องทางการติดตั้ง

**ระบบ centos**

1. ติดตั้ง libevent-devel ที่ extension event ต้องการ
```shell
yum install libevent-devel -y
# หากไม่สามารถติดตั้ง กรุณาลองใช้คำสั่งด้านล่าง
# yum install libevent2-devel -y
```
2. ติดตั้ง extension event
(extension event จำเป็นต้องใช้ PHP>=5.4)
```shell
pecl install event
```
โปรดระวังการใส่ข้อความ:```Include libevent OpenSSL support [yes] :``` เมื่อได้รับคำถาม ให้ป้อน ```no``` และกด Enter  เพื่อดำเนินการต่อ ต่อไปเพื่อดำเนินการติดตั้ง

3. รันคำสั่ง ```php --ini``` เพื่อค้นหาและเปิดไฟล์ php.ini และเพิ่มคอนฟิกเรเช็นชั่นต่อไปนี้ไว้ที่บรรทัดสุดท้าย
```shell
extension=event.so
```

**ระบบ Debian / Ubuntu**

1. ติดตั้ง libevent-dev ที่ extension event ต้องการ
```shell
apt-get install libevent-dev -y
# หากไม่สามารถสติดตั้ง กรุณาลองใช้คำสั่งด้านล่าง
# apt-get install libevent2-dev -y
```
2. ติดตั้ง extension event
```shell
pecl install event
```
โปรดระวังการใส่ข้อความ:```Include libevent OpenSSL support [yes] :``` เมื่อได้รับคำถาม ให้ป้อน ```no``` และกด Enter  เพื่อดำเนินการต่อ ต่อไปเพื่อดำเนินการติดตั้ง

3. รันคำสั่ง ```php --ini``` เพื่อค้นหาและเปิดไฟล์ php.ini และเพิ่มคอนฟิกเรเช็นชั่นต่อไปนี้ไว้ที่บรรทัดสุดท้าย
```shell
extension=event.so
```

**แนะนำการติดตั้งบนระบบหลักของ Mac OS**
ปกติแล้ว Mac OS จะถูกใช้เป็นเครื่องพัฒนา สามารถไม่ต้องติดตั้ง extension event ได้

# การติดตั้งระบบใหม่ (การติดตั้ง PHP ทั้งหมด+ extension)

## วิธีการติดตั้งบนระบบ centos
1. ดำเนินการติดตั้งผ่าน command line (ขั้นตอนนี้รวมถึงการติดตั้งโปรแกรมหลักของ php-cli และเฉพาะ pcntl, posix, libevent library  และ git)
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```
2. ติดตั้ง extension event
(หมายเหตุ: extension event จำเป็นต้องใช้ PHP>=5.4)
```shell
pecl install event
```
โปรดระวังการใส่ข้อความ:```Include libevent OpenSSL support [yes] :``` เมื่อได้รับคำถาม ให้ป้อน ```no``` และกด Enter  เพื่อดำเนินการต่อ ต่อไปเพื่อดำเนินการติดตั้ง
3. รันคำสั่ง ```php --ini``` เพื่อค้นหาและเปิดไฟล์ php.ini และเพิ่มคอนฟิกเรเช็นชั่นต่อไปนี้ไว้ที่บรรทัดสุดท้าย
```shell
extension=event.so
```
4. ดำเนินการ clone WorkerMan main program ผ่าน GitHub
```shell
git clone https://github.com/walkor/Workerman
```
5. สามารถอ้างอิงแนวทาง[Getting Started- Simple Development Examples](../getting-started/simple-example.md) หรือดาวน์โหลดระบบตัวอย่างที่ถูกแพ็คมาจาก[เว็บไซต์อย่างเป็นทางการ](https://www.workerman.net/) และทำการรันได้
## คู่มือการติดตั้งบนระบบ debian/ubuntu

1. ให้รันคำสั่งต่อไปนี้ใน Command Line (ขั้นตอนนี้รวมถึงการติดตั้งโปรแกรมหลักของ PHP-cli, ไลบรารี libevent และ git)
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2. ติดตั้ง extension event โดยรันคำสั่งนี้ใน Command Line
(โปรดทราบ: extension event ต้องการ PHP>=5.4)
```shell
pecl install event
```
หลังจากบอกให้```Include libevent OpenSSL support [yes] :``` ให้พิมพ์ ```no``` และกด Enter, ไม่ต้องกระโดดไปทางอื่น

3. ให้รันคำสั่ง ```php --ini``` เพื่อค้นหาและเปิดไฟล์ php.ini และเพิ่มค่าต่อไปนี้ในบรรทัดสุดท้าย
```shell
extension=event.so
```

4. รันคำสั่งต่อไปนี้ใน Command Line (ขั้นตอนนี้เป็นการดาวน์โหลดโปรแกรมหลักของ WorkerMan จาก github)
```shell
git clone https://github.com/walkor/Workerman
```

5. อ่าน [หัวข้อเริ่มต้น - ตัวอย่างการพัฒนาเบื้องต้น](../getting-started/simple-example.md) เพื่อเขียนไฟล์ทางเข้าและรัน
หรือดาวน์โหลดตัวอย่างพร้อมใช้จาก [เว็บไซต์อย่างเป็นทางการ](https://www.workerman.net/)

## คู่มือการติดตั้งบนระบบ mac os
**วิธีที่ 1:** mac os มี PHP Cli มาพร้อมในระบบ แต่อาจขาด extension ```pcntl``` 

1. ดูประกาศ [เพิ่มเติม-การติดตั้ง extension](../appendices/install-extension.md) ในส่วนการติดตั้ง ```pcntl``` จากรหัสผลิตภัณฑ์

2. ดูประกาศ [เพิ่มเติม-การติดตั้ง extension](../appendices/install-extension.md) ในส่วนการติดตั้ง ```event``` โดยใช้ phpize (คำสั่งนี้สามารถข้ามได้ถ้าเป็นเครื่องพัฒนา)

3. ดาวน์โหลดโปรแกรมหลักของ WorkerMan จาก https://www.workerman.net/download/workermanzip  หรือจาก [เว็บไซต์อย่างเป็นทางการ](https://www.workerman.net/)

**วิธีที่ 2:** ใช้คำสั่ง ```brew``` เพื่อติดตั้ง php และ extension ที่เกี่ยวข้อง

1. ให้รันคำสั่งต่อไปนี้ใน Command Line เพื่อติดตั้งเครื่องมือ ```brew``` (ถ้าได้ติดตั้ง ```brew``` ไว้แล้วสามารถข้ามขั้นตอนนี้ได้)
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. ให้รันคำสั่งต่อไปนี้ใน Command Line เพื่อติดตั้ง ```php```
```shell
brew install php
```

3. ให้รันคำสั่งต่อไปนี้ใน Command Line เพื่อติดตั้ง ```event``` extension
```shell
brew install php-event
```

4. ดาวน์โหลดตัวอย่างพร้อมใช้จาก [เว็บไซต์อย่างเป็นทางการ](https://www.workerman.net/)

# คำอธิบายเกี่ยวกับ Event Extension
[Extension Event](https://php.net/manual/zh/book.event.php) ไม่จำเป็นต้องใช้ ถ้าธุรกิจต้องการรองรับการเชื่อมต่อพร้อมกันมากกว่า 1000 การติดต่อ การแนะนำคือการติดตั้ง Event เพื่อรองรับการเชื่อมต่อพร้อมกันมากมาย หากธุรกิจต้องการรองรับการเชื่อมต่อที่น้อยเช่น 1000 การติดต่อขึ้นไป อาจจะไม่จำเป็นต้องติดตั้ง

## ปัญหาที่พบบ่อย
1. ถ้าพบข้อผิดพลาดดังต่อไปนี้ `checking for include/event2/event.h... not found`  โปรดลองลบไลบรารี libevent-dev(el) และติดตั้ง libevent2-dev(el) อีกครั้ง
สำหรับระบบ centos: yum remove libevent-devel && yum install libevent2-devel
สำหรับระบบ debian/ubuntu: apt-get remove libevent-dev && apt-get install libevent2-dev

2. ถ้าพบข้อผิดพลาดดังต่อไปนี้`NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`
โปรดเปลี่ยนลำดับการโหลดของ event.so และ socket.so นั่นคือใน php.ini ให้ตั้งค่า `extension=socket.so` ก่อน `extension=event.so` เพื่อให้ socket extension โหลดก่อน
