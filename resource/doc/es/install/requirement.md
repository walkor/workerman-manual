# Requisitos del entorno

## Usuarios de Windows
Desde la versión 3.5.3, Workerman puede admitir simultáneamente sistemas Linux y Windows.

1. Se requiere PHP >= 5.4 y configurar correctamente las variables de entorno de PHP.

2. La versión de Windows de Workerman no depende de ninguna extensión.

3. Para la instalación y limitaciones de uso, haz clic [**aquí**](https://www.workerman.net/windows).

4. Debido a las limitaciones de uso de Workerman en Windows, se recomienda utilizar sistemas Linux para entornos de producción, y Windows solo se recomienda para entornos de desarrollo.

``` ====Esta página solo es aplicable a usuarios de Linux, ignora esto si eres un usuario de Windows. ====```


## Usuarios de Linux (incluyendo Mac OS)
Los usuarios de Linux solo pueden utilizar la versión de Workerman para Linux.

1. Instala PHP >= 5.4 y las extensiones pcntl y posix.

2. Se recomienda instalar la extensión event, pero no es obligatorio (ten en cuenta que la extensión event requiere PHP >= 5.4).

### Script de verificación del entorno en Linux
Los usuarios de Linux pueden ejecutar el siguiente script para comprobar si su entorno local cumple con los requisitos de WorkerMan.

```curl -Ss https://www.workerman.net/check | php```

Si el script muestra "ok" en todas las notificaciones, significa que el entorno de ejecución de WorkerMan está satisfecho.

(Nota: El script de verificación no incluye la comprobación de la extensión event. Si el número de conexiones simultáneas es superior a 1024, se recomienda instalar la extensión event. Consulta la siguiente sección para conocer el método de instalación).

## Detalles

### Sobre PHP-CLI

WorkerMan se ejecuta en modo de línea de comandos de PHP (PHP-CLI). PHP-CLI es un programa ejecutable independiente de PHP-FPM o el módulo PHP de Apache, no entra en conflicto ni depende de ellos.

### Extensiones requeridas por WorkerMan

1. [Extensión pcntl](https://www.php.net/manual/es/book.pcntl.php)

La extensión pcntl es importante para el control de procesos en entornos Linux en PHP. WorkerMan utiliza características como la [creación de procesos](https://www.php.net/manual/es/function.pcntl-fork.php), [control de señales](https://www.php.net/manual/es/function.pcntl-signal.php), [temporizadores](https://www.php.net/manual/es/function.pcntl-alarm.php) y [monitorización del estado de procesos](https://www.php.net/manual/es/function.pcntl-waitpid.php). Esta extensión no es compatible con la plataforma Windows.

2. [Extensión posix](https://www.php.net/manual/es/book.posix.php)

La extensión posix permite a PHP en entornos Linux llamar a las interfaces proporcionadas por el estándar [POSIX](https://es.wikipedia.org/wiki/POSIX). WorkerMan utiliza principalmente estas interfaces para implementar la demonización y el control de grupos de usuarios. Esta extensión no es compatible con la plataforma Windows.

3. [Extensión Event](https://www.php.net/manual/es/book.event.php) o [Extensión libevent](https://www.php.net/manual/es/book.libevent.php)

La extensión event permite que PHP utilice mecanismos avanzados de manejo de eventos como [Epoll](https://es.wikipedia.org/wiki/Epoll) y Kqueue del sistema, lo que puede aumentar significativamente la utilización de la CPU por WorkerMan en conexiones de alto rendimiento. Es vital en aplicaciones con conexiones simultáneas de alta carga. La extensión libevent (o event) no es obligatoria; si no está instalada, se utilizará por defecto el mecanismo de manejo de eventos nativo de PHP, Select.

## Cómo instalar las extensiones

Consulta la sección de [Instalación de extensiones](../appendices/install-extension.md)
