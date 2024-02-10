# エクステンションのインストール
## 注意
Apache+PHPやNginx+PHPの実行モードとは異なり、WorkerManはPHPコマンドライン[PHP CLI](http://php.net/manual/zh/features.commandline.php)を使用しており、異なるPHP実行ファイルを使用し、異なるphp.iniファイルが使用される可能性があります。そのため、Webページで```phpinfo()```をプリントして、あるエクステンションがインストールされているのを見たとしても、コマンドラインのPHP CLIにそのエクステンションがインストールされているとは限りません。

## PHP CLIにインストールされたエクステンションの確認方法
```php -m```を実行すると、コマンドラインのPHP CLIにインストールされたエクステンションがリストされます。結果は以下のようになります：
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## PHP CLI のphp.iniファイルの場所の確認方法
エクステンションをインストールする際は、php.iniファイルを手動で構成する必要があることがあります。そのため、PHP CLIのphp.iniファイルの場所を確認する必要があります。```php --ini```を実行して、PHP CLIのiniファイルの場所を検索します。結果は次のようになります（各システムで結果は異なります）：
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

# PHP CLIにエクステンションをインストールする（memcachedエクステンションをインストールする例）
## 方法1: aptやyumコマンドを使用してインストールする
PHPがaptやyumコマンドを使用してインストールされている場合は、エクステンションもaptやyumを使用してインストールすることができます。

**debian/ubuntuなどのシステムでのaptによるPHPエクステンションのインストール方法（rootユーザーでない場合はsudoコマンドを追加する必要があります）**

1. ```apt-cache search```を使用してエクステンションパッケージを検索します
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2. ```apt-get install```を使用してエクステンションパッケージをインストールします
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**centosなどのシステムでのyumによるPHPエクステンションのインストール方法**

1. ```yum search```を使用してエクステンションパッケージを検索します
```shell
~# yum search memcached php
php-pecl-memcached - memcached module for php5
```
2. ```yum install```を使用してエクステンションパッケージをインストールします
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```
**注：**
aptやyumを使用してPHPエクステンションをインストールすると、php.iniファイルが自動的に構成されるため、インストール後すぐに使用することができますが、欠点としては、aptやyumに対応するエクステンションのパッケージがない場合があります。

## 方法2: peclを使用してインストールする
```pecl install```コマンドを使用してエクステンションをインストールします。

1. ```pecl install```を使用してインストールします
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. php.iniを構成します

```php --ini```を実行してphp.iniファイルの場所を検索し、ファイルに```extension=memcached.so```を追加します。

## 方法3: ソースコードからのコンパイルインストール（通常はPHPに同梱されているエクステンションのインストール、pcntlエクステンションをインストールする例）
1. ```php -v```コマンドを使用して現在のPHP CLIのバージョンを確認します
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. バージョンに応じてPHPソースコードをダウンロードします

PHPの過去のバージョンのダウンロードページ：https://php.net/releases/

3. ソースコードの圧縮ファイルを展開します

例えば、ダウンロードした圧縮ファイルの名前が```php-5.3.29.tar.gz```の場合
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. ext/pcntlディレクトリに移動します
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. ```phpize```コマンドを実行します
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. ```configure```コマンドを実行します
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. ```make```コマンドを実行します
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. ```make install```コマンドを実行します
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. php.iniファイルを構成します

```php --ini```を実行してphp.iniファイルの場所を検索し、ファイルに```extension=pcntl.so```を追加します。

**注：**
この方法は通常、PHPに同梱されているエクステンションをインストールする際に使用されます。```phpize```を使用して特定のエクステンションをコンパイルする他、PHP全体を再度コンパイルし、コンパイル時にエクステンションを追加することもできます。例えば、ソースコードのルートディレクトリで以下を実行します
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## 方法4: phpizeを使用してインストールする
インストールするエクステンションがPHPソースコードのextディレクトリに含まれていない場合、そのエクステンションはhttps://pecl.php.net から検索してダウンロードする必要があります

libeventエクステンションをインストールする例（システムにlibevent-devライブラリがインストールされていると仮定）

1. libeventエクステンションファイルの圧縮ファイルをダウンロードします（どのディレクトリにダウンロードするかは任意です）
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
2. エクステンションファイルの圧縮ファイルを展開します
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```
3. ソースコードのディレクトリに移動します
```shell
~# cd libevent-0.1.0/
```
4. ```phpize```コマンドを実行します
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
5. ```configure```コマンドを実行します
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```
6. ```make```コマンドを実行します
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```
7. ```make install```コマンドを実行します
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
8. php.iniファイルを構成します

```php --ini```を実行してphp.iniファイルの場所を検索し、ファイルに```extension=libevent.so```を追加します。
