# Cambios que no surten efecto

**Motivo:**

Workerman se ejecuta en memoria de forma constante, lo que permite evitar la lectura repetida desde el disco y el reanálisis y recompilación de PHP para lograr el máximo rendimiento. Por lo tanto, después de realizar cambios en el código de negocio, es necesario recargar o reiniciar manualmente para que surtan efecto.

Además, Workerman ofrece un servicio de monitoreo de archivos actualizados que detecta cualquier modificación en los archivos y luego ejecuta automáticamente una recarga, volviendo a cargar los archivos de PHP. Los desarrolladores pueden incorporar este servicio en sus proyectos para que se inicie junto con el proyecto.

Nota: Windows no admite la recarga y no puede utilizar el servicio de monitoreo.

**Direcciones de descarga del servicio de monitoreo de archivos:**

1. Versión sin dependencias: https://github.com/walkor/workerman-filemonitor

2. Versión con dependencia de inotify: https://github.com/walkor/workerman-filemonitor-inotify

**Diferencias entre las dos versiones:**

La versión en la dirección 1 utiliza un método para verificar el tiempo de actualización del archivo cada segundo con el fin de determinar si el archivo ha sido actualizado.

La versión en la dirección 2 hace uso del mecanismo de[inotify](https://baike.baidu.com/view/2645027.htm) del núcleo de Linux, el cual notifica al sistema cuando un archivo se actualiza.

Por lo general, la versión sin dependencias de la dirección 1 es suficiente.
