# Componente de monitorización de archivos

**Antecedentes:**

Workerman es un servidor en memoria residente, lo que evita leer repetidamente el disco y compilar PHP repetidamente para lograr el máximo rendimiento. Por lo tanto, los cambios en el código empresarial requieren una recarga o reinicio manual para que surtan efecto.

Al mismo tiempo, Workerman proporciona un servicio de monitorización de archivos que detecta actualizaciones de archivos y ejecuta automáticamente una recarga cuando detecta un cambio, volviendo a cargar los archivos PHP. Los desarrolladores pueden agregar este servicio a su proyecto y se iniciará junto con el proyecto.

**Dirección de descarga del servicio de monitorización de archivos:**

1. Versión sin dependencias: https://github.com/walkor/workerman-filemonitor
2. Versión de dependencia inotify: https://github.com/walkor/workerman-filemonitor-inotify (requiere la instalación de la [extensión inotify](https://php.net/manual/zh/book.inotify.php))

**Diferencias entre las dos versiones:**

La versión en la dirección 1 utiliza la técnica de sondeo del tiempo de modificación del archivo cada segundo para determinar si el archivo ha sido actualizado, mientras que la versión en la dirección 2 utiliza el mecanismo del kernel de Linux [inotify](https://baike.baidu.com/view/2645027.htm), que notifica activamente a Workerman cuando se actualiza un archivo.

Por lo general, la versión sin dependencias en la dirección 1 es suficiente.

**Forma de uso:**

Simplemente copia el directorio Applications/FileMonitor en el directorio Applications de tu proyecto.

Si tu proyecto no tiene un directorio Applications, puedes copiar el archivo Applications/FileMonitor/start.php a cualquier ubicación de tu proyecto y requerirlo en tu script de inicio.

El componente de monitorización por defecto supervisa el directorio Applications. Si necesitas cambiarlo, puedes modificar la variable `$monitor_dir` en Applications/FileMonitor/start.php, lo más recomendable es usar una ruta absoluta para el valor de `$monitor_dir`.

**Nota:**

* El sistema Windows no es compatible con la recarga, por lo que no se puede usar este servicio de monitorización.
* Solo funciona en modo de depuración. No se ejecutará la monitorización de archivos en modo daemon (las razones por las cuales no se admite el modo daemon se explican a continuación).
* Solo los archivos cargados después de que Worker::runAll se haya ejecutado pueden actualizarse en caliente, es decir, solo los archivos cargados en las devoluciones de llamada onXXX se pueden actualizar en caliente.

**¿Por qué no se admite el modo daemon?**

El modo daemon es generalmente utilizado para entornos de producción en vivo. En versiones de producción, generalmente se publican múltiples archivos y estos archivos pueden depender entre sí. Debido a que llevará cierto tiempo sincronizar múltiples archivos en el disco, es posible que en cierto momento no todos los archivos estén presentes en el disco. Si en ese momento se detecta una actualización de archivos y se ejecuta una recarga, existe el riesgo de errores fatales debido a archivos faltantes.

Además, a veces en entornos de producción es necesario solucionar errores en línea. Si se edita y guarda el código directamente, este se aplicará de inmediato, lo que podría generar errores sintácticos y dejar los servicios en línea inutilizables. El método correcto sería guardar el código, verificar si hay errores sintácticos con `php -l yourfile.php` y luego recargar el código para actualizarlo en caliente.

Si un desarrollador realmente necesita habilitar la monitorización de archivos y la actualización automática en modo daemon, puede modificar el código en Applications/FileMonitor/start.php eliminando la condición de Worker::$daemonize.
