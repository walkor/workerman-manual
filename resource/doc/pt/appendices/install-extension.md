# Instalação de Extensões
## Atenção
Diferente do modo de execução Apache+PHP ou Nginx+PHP, o WorkerMan é executado com base no PHP de linha de comando [PHP CLI](http://php.net/manual/zh/features.commandline.php), utilizando um executável PHP diferente e um arquivo php.ini também diferente. Portanto, só porque uma extensão está instalada e aparece no ```phpinfo()``` do navegador, não significa que a extensão correspondente esteja instalada para o PHP CLI.

## Como verificar as extensões instaladas para o PHP CLI
Executar o comando ```php -m``` listará as extensões já instaladas para o PHP CLI, e o resultado será semelhante ao seguinte:
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## Como verificar a localização do arquivo php.ini do PHP CLI
Ao instalar extensões, pode ser necessário configurar manualmente o arquivo php.ini, para adicionar a extensão. É necessário verificar a localização do arquivo php.ini do PHP CLI, o que pode ser feito executando o comando ```php --ini```, e o resultado será semelhante ao seguinte (os resultados variam entre os sistemas):
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

# Instalando uma extensão para o PHP CLI (instalando a extensão memcached como exemplo)
## Método 1: Usando apt ou yum para instalar
Se o PHP foi instalado usando os comandos apt ou yum, a extensão também pode ser instalada usando esses mesmos comandos.

**Método de instalação da extensão PHP através do apt em sistemas debian/ubuntu (usuários não root precisam adicionar o comando sudo)**

1. Use o comando ```apt-cache search``` para procurar o pacote de extensão
```shell
~# apt-cache search memcached php
php-apc - Módulo APC (Alternative PHP Cache) para PHP 5
php5-memcached - Módulo memcached para php5
```
2. Use o comando ```apt-get install``` para instalar o pacote de extensão
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**Para sistemas centos e outros, use yum para instalar a extensão PHP**

1. Use o comando ```yum search``` para procurar o pacote de extensão
```shell
~# yum search memcached php
php-pecl-memcached - Módulo memcached para php5
```
2. Use o comando ```yum install``` para instalar o pacote de extensão
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```
**Nota:**
Ao usar apt ou yum para instalar extensões do PHP, a configuração do arquivo php.ini é automaticamente feita após a instalação, sendo assim, a extensão estará pronta para uso imediatamente. No entanto, algumas extensões podem não ter um pacote correspondente no apt ou yum.

## Método 2: Usando pecl para instalar
Use o comando ```pecl install``` para instalar a extensão

1. Instalação utilizando ```pecl install```
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. Configuração do php.ini
Use o comando ```php --ini``` para encontrar a localização do arquivo php.ini e adicione ```extension=memcached.so``` a esse arquivo.

## Método 3: Compilação e instalação a partir do código-fonte (geralmente usado para instalar extensões fornecidas com o PHP, onde pcntl é usado como exemplo)
1. Use o comando ```php -v``` para verificar a versão atual do PHP CLI
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. Faça o download do código-fonte do PHP com base na versão

Página de download de versões antigas do PHP: https://php.net/releases/

3. Descompacte o arquivo fonte

Se o nome do arquivo baixado for ```php-5.3.29.tar.gz```, por exemplo:
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. Acesse o diretório ext/pcntl no código-fonte
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. Execute o comando ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. Execute o comando ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. Execute o comando ```make```
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. Execute o comando ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. Configurar o arquivo ini

Use o comando ```php --ini``` para encontrar a localização do arquivo php.ini e adicione ```extension=pcntl.so``` a esse arquivo.

**Nota:**
Esse método é geralmente usado para instalar extensões fornecidas com o PHP, como extensões posix e pcntl. Além de usar o phpize para compilar uma determinada extensão, também é possível recompilar todo o PHP, adicionando extensões como parâmetros durante a compilação, por exemplo:
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## Método 4: Instalando usando phpize
Se a extensão que deseja instalar não estiver presente no diretório ext do código-fonte do PHP, será necessário pesquisar e baixar a extensão em https://pecl.php.net, como exemplo de instalação da extensão libevent (supondo que o sistema tenha a biblioteca libevent-dev instalada):

1. Baixe o arquivo compactado da extensão libevent (de qualquer diretório no sistema)
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

2. Descompacte o arquivo compactado da extensão
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3. Acesse o diretório de origem
```shell
~# cd libevent-0.1.0/
```

4. Execute o comando ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

5. Execute o comando ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```

6. Execute o comando ```make```
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7. Execute o comando ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```

8. Configurar o arquivo ini

Use o comando ```php --ini``` para encontrar a localização do arquivo php.ini e adicione ```extension=libevent.so``` a esse arquivo.
