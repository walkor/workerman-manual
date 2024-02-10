# 安裝擴展
## 注意
與Apache+PHP或者Nginx+PHP的運行模式不同，WorkerMan是基於PHP命令行 [PHP CLI](http://php.net/manual/zh/features.commandline.php) 運行的，使用的是不同的PHP可執行程序，使用的php.ini文件也可能不同。所以在網頁中打印```phpinfo()```看到安裝了某個擴展，不代表命令行的PHP CLI也安裝了對應的擴展。

## 如何確定PHP CLI安裝了哪些擴展
運行 ```php -m``` 會列出命令行 PHP CLI 已經安裝的擴展，結果類似如下：
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## 如何確定PHP CLI 的php.ini文件的位置
當我們安裝擴展時，可能需要手動配置php.ini文件，把擴展加進去，所以要確認PHP CLI的php.ini文件的位置。可以運行```php --ini```查找PHP CLI的ini文件位置，結果類似如下(各個系統顯示結果會有差異)：
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

# 給PHP CLI安裝擴展（安裝memcached擴展為例）
## 方法一、使用apt或者yum命令安裝
如果PHP是通過 apt 或者 yum 命令安裝的，則擴展也可以通過 apt 或者 yum 安裝

**debian/ubuntu等系統apt安裝PHP擴展方法（非root用戶需要加sudo命令）**

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

**centos等系統yum安裝PHP擴展方法**

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

使用apt或者yum安裝PHP擴展會自動配置php.ini文件，安裝完直接可用，十分方便。缺點是有些擴展在apt或者yum中沒有對應的擴展安裝包。

## 方法二、使用pecl安裝
使用```pecl install```命令安裝擴展

1、```pecl install```安裝
```
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2、配置php.ini

通過運行 ```php --ini```查找php.ini文件位置，然後在文件中添加```extension=memcached.so```

## 方法三、源碼編譯安裝（一般是安裝PHP自帶的擴展，以安裝pcntl擴展為例）
1、利用```php -v```命令查看當前的PHP CLI的版本
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2、根據版本下載PHP源代碼

PHP歷史版本下載頁面：https://php.net/releases/

3、解壓源碼壓縮包

例如下載的壓縮包名稱是```php-5.3.29.tar.gz```
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4、進入源碼中的ext/pcntl目錄
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
9、配置ini文件

通過運行 ```php --ini```查找php.ini文件位置，然後在文件中添加```extension=pcntl.so```

**說明：**
此方法一般用來安裝PHP自帶的擴展，例如posix擴展和pcntl擴展。除了用phpize編譯某個擴展，也可以重新編譯整個PHP，在編譯時用參數添加擴展，例如在源碼根目錄運行
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## 方法四、phpize安裝
如果要安裝的擴展在php源碼ext目錄中沒有，那麼這個擴展需要到https://pecl.php.net 搜索下載

以安裝libevent擴展為例（假設系統安裝了libevent-dev庫）

1、下載libevent擴展文件壓縮包（在當前系統哪個目錄下載隨意）
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

8、配置ini文件

通過運行 ```php --ini```查找php.ini文件位置，然後在文件中添加```extension=libevent.so```
