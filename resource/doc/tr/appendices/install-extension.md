# Kurulumu Genişletme
## Dikkat
Apache+PHP veya Nginx+PHP çalışma modundan farklı olarak, WorkerMan PHP komut satırı [PHP CLI](http://php.net/manual/zh/features.commandline.php) üzerinde çalışır ve farklı PHP yürütülebilir dosyasını kullanır, ayrıca farklı bir php.ini dosyası kullanılabilir. Bu nedenle, bir websitesinde ```phpinfo()``` çağrısı ile bir eklentinin yüklü olduğunu görmek, komut satırı PHP CLI'nin aynı eklentiyi yüklediği anlamına gelmez.

## PHP CLI'nin hangi eklentilerin yüklü olduğunu nasıl belirleriz
```php -m``` komutunu çalıştırarak komut satırı PHP CLI'nin yüklü eklentileri listelenir, sonuç aşağıdakine benzer olacaktır:
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## PHP CLI'nin php.ini dosyasının konumu nasıl belirlenir
Eklenti kurulumu sırasında, php.ini dosyasını elle yapılandırmamız gerekebilir, bu nedenle PHP CLI'nin php.ini dosyasının konumunu 'php --ini' komutunu çalıştırarak bulabiliriz, sonuç aşağıdakine benzer olacaktır (farklı sistemlerde sonuçlar değişebilir):
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

# PHP CLI'ye Eklenti Kurma (memcached eklentisi örneği)
## Yöntem 1: apt veya yum komutunu kullanarak kurma
Eğer PHP apt veya yum komutlarıyla yüklendiyse, genişletmeler de aynı şekilde apt veya yum ile yüklenebilir

**debian/ubuntu ve benzeri sistemlerde apt ile PHP eklentisi kurma (root olmayan kullanıcılar sudo komutunu eklemelidir)**

1. ```apt-cache search``` komutu ile genişletme paketini arayın
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2. ```apt-get install``` komutu ile genişletme paketini kurun
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**centos ve benzeri sistemlerde yum ile PHP eklentisi kurma**

1. ```yum search``` komutu ile genişletme paketini arayın
```shell
~# yum search memcached php
php-pecl-memcached - memcached module for php5
```
2. ```yum install``` komutu ile genişletme paketini kurun
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```
**Açıklama:**

apt veya yum ile genişletme kurulumu php.ini dosyasını otomatik olarak yapılandırır, kurulum tamamlandıktan sonra doğrudan kullanılabilir, çok pratik olmasına rağmen bazı genişletmeler için uygun paket bulunmayabilir.

## Yöntem 2: pecl ile kurma
```pecl install``` komutunu kullanarak genişletme kurma

1. ```pecl install``` komutu ile kurulum yapın
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. php.ini dosyasını yapılandırma

```php --ini``` komutu ile php.ini dosyasının konumunu bulun ve dosyaya ```extension=memcached.so``` ekleyin

## Yöntem 3: Kaynak kodu derleyerek kurma (genellikle dahili PHP genişletmeleri için, pcntl genişletmesi örneği ile)
1. Mevcut PHP CLI sürümünü görmek için ```php -v``` komutunu çalıştırın
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. Sürüme göre PHP kaynak kodunu indirin

PHP geçmiş sürümleri indirme sayfası: https://php.net/releases/

3. Kaynak kod sıkıştırılmış dosyasını açın

Örneğin, indirilen dosyanın adı ```php-5.3.29.tar.gz``` olsun
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. ext/pcntl klasörüne gidin
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. ```phpize``` komutunu çalıştırın
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. ```configure``` komutunu çalıştırın
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. ```make``` komutunu çalıştırın
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. ```make install``` komutunu çalıştırın
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. php.ini dosyasını yapılandırma

```php --ini``` komutu ile php.ini dosyasının konumunu bulun ve dosyaya ```extension=pcntl.so``` ekleyin

**Açıklama:**
Bu yöntem genellikle dahili PHP genişletmeleri için kullanılır, örneğin posix genişletmesi ve pcntl genişletmesi için. Bir genişletmeyi phpize ile derlemek dışında, tüm PHP'yi yeniden derleyerek parametrelerle genişletmeler ekleyebilirsiniz, örneğin kaynak kodun kök dizininde aşağıdaki gibi çalıştırın:
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## Yöntem 4: phpize ile kurma
Eğer kurmak istediğiniz genişletme php kaynak kodu ext klasöründe bulunmuyorsa, bu genişletmeyi https://pecl.php.net adresinden arayarak indirmeniz gerekmektedir.

Libevent genişletmesi örneği ile kurulum yapalım (sistemin libevent-dev kütüphanesinin kurulu olduğunu varsayalım)

1. Libevent genişletme dosyasını indirin (herhangi bir dizine indirebilirsiniz)
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
2. Genişletme dosyasını açın
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```
3. Genişletme dizinine gidin
```shell
~# cd libevent-0.1.0/
```
4. ```phpize``` komutunu çalıştırın
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
5. ```configure``` komutunu çalıştırın
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```
6. ```make``` komutunu çalıştırın
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```
7. ```make install``` komutunu çalıştırın
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
8. php.ini dosyasını yapılandırma

```php --ini``` komutu ile php.ini dosyasının konumunu bulun ve dosyaya ```extension=libevent.so``` ekleyin
