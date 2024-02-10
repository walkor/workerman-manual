# Instalación de extensiones
## Nota
Diferente al modo de operación de Apache+PHP o Nginx+PHP, WorkerMan se ejecuta en modo de línea de comandos de PHP [PHP CLI](http://php.net/manual/zh/features.commandline.php), y utiliza un ejecutable de PHP diferente, por lo que es posible que también utilice un archivo php.ini diferente. Por lo tanto, ver una extensión instalada al imprimir ```phpinfo()``` en una página web no significa necesariamente que esta extensión también esté instalada para PHP CLI en la línea de comandos.

## Cómo determinar qué extensiones están instaladas para PHP CLI
Ejecutando ```php -m``` se listarán las extensiones instaladas para PHP CLI. El resultado será similar al siguiente:
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## Cómo determinar la ubicación del archivo php.ini de PHP CLI
Cuando se instalan extensiones, puede ser necesario configurar manualmente el archivo php.ini, agregando la extensión. Para confirmar la ubicación del archivo php.ini de PHP CLI, se puede ejecutar ```php --ini``` para buscar la ubicación del archivo. El resultado será similar al siguiente (los resultados variarán según el sistema):
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

# Instalación de extensiones para PHP CLI (Instalación de la extensión memcached como ejemplo)
## Método uno: Usar apt o yum para instalar
Si PHP ha sido instalado usando los comandos apt o yum, las extensiones también pueden ser instaladas usando apt o yum.

**Método de instalación de la extensión PHP en sistemas debian/ubuntu (los usuarios no root deben añadir el comando sudo)**
1. Usar ```apt-cache search``` para buscar el paquete de la extensión
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2. Usar ```apt-get install``` para instalar el paquete de la extensión
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...

**Nota:**
La instalación de la extensión PHP a través de apt o yum configurará automáticamente el archivo php.ini, por lo que estará lista para usar. Sin embargo, el inconveniente es que algunas extensiones pueden no tener un paquete de instalación correspondiente en apt o yum.

## Método dos: Instalar usando pecl
Usar el comando ```pecl install``` para instalar la extensión

1. Instalación con ```pecl install```
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. Configurar php.ini

Usar el comando ```php --ini``` para buscar la ubicación del archivo php.ini y luego añadir ```extension=memcached.so``` al archivo.

## Método tres: Instalación mediante compilación del código fuente (Generalmente para instalar extensiones incluidas en PHP, usando pcntl como ejemplo)
1. Usar el comando ```php -v``` para ver la versión actual de PHP CLI
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. Descargar el código fuente de PHP según la versión

Página de descarga de versiones históricas de PHP: https://php.net/releases/

3. Descomprimir el archivo comprimido
Por ejemplo, si el archivo comprimido descargado se llama ```php-5.3.29.tar.gz```
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. Ingresar al directorio ext/pcntl del código fuente
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. Ejecutar el comando ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. Ejecutar el comando ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. Ejecutar el comando ```make```
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. Ejecutar el comando ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. Configurar el archivo ini

Usar el comando ```php --ini``` para buscar la ubicación del archivo php.ini, luego añadir ```extension=pcntl.so``` al archivo.

**Nota:**
Este método generalmente se utiliza para instalar extensiones incluidas en PHP, como pcntl y posix. Además de compilar una extensión con phpize, también se puede recompilar PHP completo agregando extensiones como argumentos durante la compilación, por ejemplo ejecutando el siguiente comando en el directorio raíz del código fuente
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## Método cuatro: Instalación mediante phpize
Si la extensión que se desea instalar no se encuentra en el directorio ext del código fuente de PHP, entonces es necesario buscarla y descargarla desde https://pecl.php.net

Usaremos la instalación de la extensión libevent como ejemplo (asumiendo que el sistema ya tiene la biblioteca libevent-dev instalada)
1. Descargar el archivo comprimido de la extensión libevent (desde cualquier directorio en el sistema)
```shell
~# wget https://pecl.php.net/get/libevent-0.1.0.tgz
--2015-05-26 21:43:40--  https://pecl.php.net/get/libevent-0.1.0.tgz
Resolving pecl.php.net... 104.236.228.160
Connecting to pecl.http.net|104.236.228.160|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9806 (9.6K) [application/octet-stream]
Saving to: “libevent-0.1.0.tgz”

100%[=======================================================>] 9,806       41.4K/s   in 0.2s

```
2. Descomprimir el archivo
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```
3. Ingresar al directorio del código fuente
```shell
~# cd libevent-0.1.0/
```
4. Ejecutar el comando ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
5. Ejecutar el comando ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```
6. Ejecutar el comando ```make```
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```
7. Ejecutar el comando ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
8. Configurar el archivo ini

Usar el comando ```php --ini``` para buscar la ubicación del archivo php.ini, luego añadir ```extension=libevent.so``` al archivo.
