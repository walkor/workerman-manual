# Cài đặt các extension
## Lưu ý
Khác với cách Apache+PHP hoặc Nginx+PHP hoạt động, Workerman hoạt động dựa trên PHP command line [PHP CLI](http://php.net/manual/zh/features.commandline.php), sử dụng chương trình thực thi PHP khác nhau, và có thể sử dụng tệp php.ini khác nhau. Do đó, khi in ```phpinfo()``` trên trang web và thấy cài đặt một extension, không có nghĩa là PHP CLI đã cài đặt được extension tương ứng.

## Làm thế nào để xác định cài đặt các extension cho PHP CLI
Chạy lệnh ```php -m``` sẽ liệt kê các extension đã được cài đặt cho PHP CLI, kết quả sẽ tương tự như sau:
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## Làm thế nào để xác định vị trí tệp php.ini của PHP CLI
Khi cài đặt các extension, có thể cần phải cấu hình tệp php.ini bằng tay để thêm extension vào, do đó cần xác định vị trí tệp php.ini của PHP CLI. Có thể chạy ```php --ini``` để tìm vị trí tệp php.ini của PHP CLI, kết quả sẽ tương tự như sau (kết quả sẽ khác nhau trên từng hệ thống):
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

# Cách cài đặt extension cho PHP CLI (cài đặt extension memcached làm ví dụ)
## Phương pháp một: Sử dụng lệnh apt hoặc yum để cài đặt
Nếu PHP được cài đặt thông qua lệnh apt hoặc yum, thì extension cũng có thể được cài đặt thông qua apt hoặc yum.

**Cách cài đặt extension PHP trên hệ thống debian/ubuntu (người dùng không phải root phải nhập lệnh sudo)**

1. Sử dụng ```apt-cache search``` để tìm kiếm gói extension
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2. Sử dụng ```apt-get install``` để cài đặt gói extension
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**Cách cài đặt extension PHP trên hệ thống centos**

1. Sử dụng ```yum search``` để tìm kiếm gói extension
```shell
~# yum search memcached php
php-pecl-memcached - memcached module for php5
```
2. Sử dụng ```yum install``` để cài đặt gói extension
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```
**Lưu ý:**

Sử dụng apt hoặc yum để cài đặt extension PHP sẽ tự động cấu hình tệp php.ini, sau khi cài đặt có thể sử dụng ngay, rất thuận tiện. Nhược điểm là một số extension không có gói cài đặt tương ứng trong apt hoặc yum.

## Phương pháp hai: Sử dụng pecl để cài đặt
Sử dụng lệnh ```pecl install``` để cài đặt extension

1. Cài đặt bằng ```pecl install```
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. Cấu hình php.ini

Sử dụng lệnh ```php --ini``` để tìm vị trí tệp php.ini, sau đó thêm dòng ```extension=memcached.so``` vào tệp php.ini

## Phương pháp ba: Cài đặt từ mã nguồn (thường cài đặt các extension đi kèm với PHP, lấy extension pcntl làm ví dụ)
1. Sử dụng lệnh```php -v``` để xem phiên bản PHP CLI hiện tại
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. Tải mã nguồn PHP dựa trên phiên bản

Trang tải lịch sử phiên bản PHP: https://php.net/releases/

3. Giải nén tệp nén mã nguồn

Ví dụ: Nếu tên tệp nén tải về là ```php-5.3.29.tar.gz```
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. Vào thư mục ext/pcntl trong mã nguồn
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. Chạy lệnh ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. Chạy lệnh ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. Chạy lệnh ```make```
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. Chạy lệnh```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. Cấu hình tệp ini

Sử dụng lệnh ```php --ini``` để tìm vị trí tệp php.ini, sau đó thêm dòng ```extension=pcntl.so``` vào tệp php.ini

**Lưu ý:**
Phương pháp này thường dùng để cài đặt các extension đi kèm với PHP, chẳng hạn như extension posix và pcntl. Ngoài việc sử dụng phpize để biên dịch một extension, cũng có thể biên dịch lại toàn bộ PHP và thêm extension bằng cách sử dụng tham số khi biên dịch, chẳng hạn khi chạy trong thư mục nguồn:
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## Phương pháp bốn: Cài đặt bằng phpize
Nếu extension cần cài đặt không có trong thư mục ext của mã nguồn PHP, extension cần phải tìm và tải về từ https://pecl.php.net

Lấy ví dụ cài đặt extension libevent (giả sử hệ thống đã cài đặt thư viện libevent-dev)

1. Tải tệp nén của extension libevent (tải tệp nén ở bất kỳ đâu trên hệ thống)
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
2. Giải nén tệp nén extension
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```
3. Vào thư mục nguồn
```shell
~# cd libevent-0.1.0/
```
4. Chạy lệnh ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
5. Chạy lệnh ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```
6. Chạy lệnh ```make```
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```
7. Chạy lệnh ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
8. Cấu hình tệp ini

Sử dụng lệnh ```php --ini``` để tìm vị trí tệp php.ini, sau đó thêm dòng ```extension=libevent.so``` vào tệp php.ini
