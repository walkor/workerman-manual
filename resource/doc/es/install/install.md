# Instrucciones de instalación
Workerman es en realidad un paquete de código PHP. Si tu entorno de PHP ya está configurado, solo necesitas descargar el código fuente o la demo de Workerman para ejecutarlo.

**Instalación mediante Composer:**
```sh
composer require workerman/workerman
```

> **Nota**
> Algunos espejos de composer pueden no estar completos. En ese caso, usa el comando `composer config -g --unset repos.packagist` para eliminar el espejo.

# Usuarios de Windows (lectura obligatoria)

A partir de la versión 3.5.3 de Workerman, ya es compatible con sistemas Windows y Linux. Los usuarios de Windows necesitan configurar las variables de entorno de PHP.

 ` === Los siguientes pasos solo son aplicables para entornos Linux de Workerman, los usuarios de Windows pueden ignorarlos === `

# Verificación del entorno del sistema Linux
El sistema Linux puede usar el siguiente script para probar si el entorno PHP local cumple con los requisitos de ejecución de Workerman.
 `curl -Ss https://www.workerman.net/check | php`

Si todas las pruebas en el script se muestran como "ok", significa que cumple con los requisitos de Workerman. Puedes descargar el ejemplo directamente desde el [sitio web oficial](https://www.workerman.net/) y ejecutarlo.

Si no todas las pruebas resultan en "ok", consulta el siguiente documento para instalar las extensiones faltantes.

(Nota: El script de prueba no verifica la extensión 'event'. Si el número de conexiones concurrentes es superior a 1024, es necesario instalar la extensión 'event' y [optimizar el kernel de Linux](../appendices/kernel-optimization.md). Consulta las instrucciones a continuación para instalar la extensión).

# Instalación de extensiones faltantes en un entorno PHP existente

## Instalación de las extensiones pcntl y posix:

**Sistema CentOS**
Si PHP se instaló a través de yum, ejecuta el siguiente comando en la línea de comandos ```yum install php-process``` para instalar las extensiones pcntl y posix.

Si la instalación falla o PHP no se instaló a través de yum, consulta el método tres de la sección [Apéndices - Instalación de extensiones](../appendices/install-extension.md) en el manual para compilar la instalación desde el código fuente.

**Sistemas Debian/Ubuntu/macOS**
Consulta el método tres de la sección [Apéndices - Instalación de extensiones](../appendices/install-extension.md) en el manual para compilar la instalación desde el código fuente.

## Instalación de la extensión 'event':
Para admitir un mayor número de conexiones concurrentes, es necesario instalar la extensión 'event' y [optimizar el kernel de Linux](../appendices/kernel-optimization.md). Sigue los siguientes pasos para instalarla:

**Sistema CentOS**

1. Instalar el paquete de desarrollo libevent-devel requerido por la extensión 'event', ejecutando el siguiente comando en la línea de comandos:
```shell
yum install libevent-devel -y
# Si no se puede instalar, prueba con el siguiente comando
# yum install libevent2-devel -y
```

2. Instalar la extensión 'event', ejecutando el siguiente comando en la línea de comandos:
(la extensión 'event' requiere PHP >= 5.4)
```shell
pecl install event
```
Ten en cuenta la indicación: ```Include libevent OpenSSL support [yes] :``` y responde con ```no``` seguido de un clic en "Enter", y luego presiona "Enter" para continuar.

3. Ejecuta ```php --ini``` para encontrar y abrir el archivo php.ini, luego agrega la siguiente configuración en la última línea:
```shell
extension=event.so
```

**Instalación en sistemas Debian/Ubuntu**

1. Instalar el paquete de desarrollo libevent-dev requerido por la extensión 'event', ejecutando el siguiente comando en la línea de comandos:
```shell
apt-get install libevent-dev -y
# Si no se puede instalar, prueba con el siguiente comando
# apt-get install libevent2-dev -y
```

2. Instalar la extensión 'event', ejecutando el siguiente comando en la línea de comandos:
```shell
pecl install event
```
Ten en cuenta la indicación: ```Include libevent OpenSSL support [yes] :``` y responde con ```no``` seguido de un clic en "Enter", y luego presiona "Enter" para continuar.

3. Ejecuta ```php --ini``` para encontrar y abrir el archivo php.ini, luego agrega la siguiente configuración en la última línea:
```shell
extension=event.so
```

**Instrucciones de instalación para sistemas macOS**

Dado que macOS generalmente se utiliza como máquina de desarrollo, no es necesario instalar la extensión 'event'.

# Instalación de un sistema completamente nuevo (instalación de PHP + extensiones)

## Instrucciones de instalación para sistemas CentOS

1. Ejecuta el siguiente comando en la línea de comandos (este paso incluye la instalación del programa principal php-cli, pcntl, posix, la biblioteca libevent y el programa git):
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2. Instala la extensión 'event', ejecutando el siguiente comando en la línea de comandos
(Nota: la extensión 'event' requiere PHP >= 5.4)
```shell
pecl install event
```
Ten en cuenta la indicación: ```Include libevent OpenSSL support [yes] :``` y responde con ```no``` seguido de un clic en "Enter", y luego presiona "Enter" para continuar.

3. Ejecuta ```php --ini``` para encontrar y abrir el archivo php.ini, luego agrega la siguiente configuración en la última línea:
```shell
extension=event.so
```

4. Ejecuta el siguiente comando en la línea de comandos (este paso implica la descarga del programa principal de Workerman a través de GitHub):
```shell
git clone https://github.com/walkor/Workerman
```

5. Consulta la sección [Guía de Introducción - Sección de Ejemplos de Desarrollo Simple](../getting-started/simple-example.md) para escribir un archivo de entrada y ejecutarlo, o descarga y ejecuta el demo desde el [sitio web oficial](https://www.workerman.net/).

## Instrucciones de instalación para sistemas Debian/Ubuntu

1. Ejecuta el siguiente comando en la línea de comandos (este paso incluye la instalación del programa principal php-cli, la biblioteca libevent y el programa git):
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2. Instala la extensión 'event', ejecutando el siguiente comando en la línea de comandos
(Nota: la extensión 'event' requiere PHP >= 5.4)
```shell
pecl install event
```
Ten en cuenta la indicación: ```Include libevent OpenSSL support [yes] :``` y responde con ```no``` seguido de un clic en "Enter", y luego presiona "Enter" para continuar.

3. Ejecuta ```php --ini``` para encontrar y abrir el archivo php.ini, luego agrega la siguiente configuración en la última línea:
```shell
extension=event.so
```

4. Ejecuta el siguiente comando en la línea de comandos (este paso implica la descarga del programa principal de Workerman a través de GitHub):
```shell
git clone https://github.com/walkor/Workerman
```

5. Consulta la sección [Guía de Introducción - Sección de Ejemplos de Desarrollo Simple](../getting-started/simple-example.md) para escribir un archivo de entrada y ejecutarlo, o descarga y ejecuta el demo desde el [sitio web oficial](https://www.workerman.net/).

## Instrucciones de instalación para sistemas macOS

**Método 1:** macOS ya viene con PHP CLI, pero es posible que le falte la extensión ```pcntl```.

1. Consulta la sección [Apéndices - Instalación de extensiones](../appendices/install-extension.md) en el manual para compilar e instalar la extensión ```pcntl``` desde el código fuente.

2. Consulta la sección [Apéndices - Instalación de extensiones](../appendices/install-extension.md) en el manual para instalar la extensión ```event``` utilizando phpize (esto se puede omitir si se utiliza como máquina de desarrollo).

3. Descarga el programa principal de Workerman desde https://www.workerman.net/download/workermanzip, o descárgalo desde el [sitio web oficial](https://www.workerman.net/) y ejecuta el ejemplo.

**Método 2:** Instala PHP y las extensiones correspondientes utilizando el comando ```brew```.

1. Ejecuta el siguiente comando en la línea de comandos para instalar la herramienta ```brew``` (si ya tienes instalado ```brew```, puedes omitir este paso)
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. Ejecuta el siguiente comando en la línea de comandos para instalar ```php```
```shell
brew install php
```

3. Ejecuta el siguiente comando en la línea de comandos para instalar la extensión ```event```
```shell
brew install php-event
```

4. Descarga y ejecuta el ejemplo desde el [sitio web oficial](https://www.workerman.net/)


# Información sobre la extensión Event
La extensión [Event](https://php.net/manual/zh/book.event.php) no es obligatoria, pero se recomienda su instalación cuando se necesitan admitir más de 1000 conexiones concurrentes, ya que puede admitir un gran número de conexiones. Si las conexiones concurrentes son bajas, por ejemplo, menos de 1000, no es necesario instalarla.

## Problemas comunes
1. Si aparece el siguiente error `checking for include/event2/event.h... not found`, intenta eliminar e instalar el paquete libevent2-dev(el) en lugar de libevent-dev(el).
Sistemas CentOS: yum remove libevent-devel && yum install libevent2-devel
Sistemas Debian/Ubuntu: apt-get remove libevent-dev && apt-get install libevent2-dev

2. Si aparece el siguiente error `NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`.
Cambia el orden de carga de 'event.so' y 'socket.so' en php.ini, es decir, coloca `extension=socket.so` antes de `extension=event.so`, para que la extensión de socket se cargue primero.
