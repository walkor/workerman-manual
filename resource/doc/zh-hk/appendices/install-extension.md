# 安裝擴展
## 注意
與 Apache+PHP 或者 Nginx+PHP 的運行模式不同，WorkerMan 是基於 PHP 命令行 [PHP CLI](http://php.net/manual/zh/features.commandline.php) 運行的，使用的是不同的 PHP 可執行程式，使用的 php.ini 文件也可能不同。所以在網頁中打印```phpinfo()```看到安裝了某個擴展，不代表命令行的 PHP CLI 也安裝了對應的擴展。

## 如何確定 PHP CLI 安裝了哪些擴展
執行 ```php -m``` 會列出命令行 PHP CLI 已經安裝的擴展，結果類似如下：
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## 如何確定 PHP CLI 的 php.ini 文件的位置
當我們安裝擴展時，可能需要手動配置 php.ini 文件，把擴展加進去，所以要確認 PHP CLI 的 php.ini 文件的位置。可以執行```php --ini```尋找 PHP CLI 的 ini 文件位置，結果類似如下(各個系統顯示結果會有差異)：
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

# 給 PHP CLI 安裝擴展（安裝 memcached 擴展為例）
## 方法一、使用 apt 或者 yum 命令安裝
如果 PHP 是通過 apt 或者 yum 命令安裝的，則擴展也可以通過 apt 或者 yum 安裝

**debian/ubuntu 等系統 apt 安裝 PHP 擴展方法（非root用戶需要加 sudo 命令）**

1、利用```apt-cache search```查找擴展包
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2、使用```apt-get install```安裝擴展包
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**centos 等系統 yum 安裝 PHP 擴展方法**

1、利用```yum search```查找擴展包
```shell
~# yum search memcached php
php-pecl-memcached - memcached module for php5
```
2、使用```yum install```安裝擴展包
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```
**說明：**

使用 apt 或者 yum 安裝 PHP 擴展會自動配置 php.ini 文件，安裝完直接可用，十分方便。缺點是有些擴展在 apt 或者 yum 中沒有對應的擴展安裝包。

## 方法二、使用 pecl 安裝
使用```pecl install```命令安裝擴展

1、```pecl install```安裝
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2、配置 php.ini

通過運行 ```php --ini```查找 php.ini 文件位置，然後在文件中添加```extension=memcached.so```

## 方法三、源碼編譯安裝（一般是安裝 PHP 自帶的擴展，以安裝 pcntl 擴展為例）
1、利用```php -v```命令查看當前的 PHP CLI 的版本
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2、根據版本下載 PHP 源代碼

PHP 歷史版本下載頁面：https://php.net/releases/

3、解壓源碼壓縮包

例如下載的壓縮包名稱是```php-5.3.29.tar.gz```
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4、進入源碼中的 ext/pcntl 目錄
```shell
~# cd php-5.3.29/ext/pcntl/
```
5、運行 ```phpize``` 命令
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6、運行 ```configure```命令
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7、運行 ```make``` 命令
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8、運 行```make install``` 命令
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9、配置 ini 文件

通過運行 ```php --ini```查找 php.ini 文件位置，然後在文件中添加```extension=pcntl.so```

**說明：**

此方法一般用來安裝 PHP 自帶的擴展，例如 posix 擴展和 pcntl 擴展。除了用 phpize 編譯某個擴展，也可以重新編譯整個 PHP，在編譯時用參數添加擴展，例如在源碼根目錄運行
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## 方法四、phpize安裝
如果要安裝的擴展在php源碼ext目錄中沒有，那麼這個擴展需要到https://pecl.php.net 搜索下載

以安裝 libevent 擴展為例（假設系統安裝了 libevent-dev 库）

1、下載 libevent 擴展文件壓縮包（在當前系統哪個目錄下載隨意）
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

2、解壓擴展文件壓縮包
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3、進入到源碼目錄
```shell
~# cd libevent-0.1.0/
```

4、運行``` phpize ```命令
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

5、運行``` configure ```命令
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```

6、運行``` make ```命令
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7、運行``` make install ```命令
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```

8、配置 ini 文件

通過運行 ```php --ini```查找 php.ini 文件位置，然後在文件中添加```extension=libevent.so```
