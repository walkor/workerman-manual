# Iniciar y detener

Tenga en cuenta que los comandos de inicio y detención de Workerman se realizan en la línea de comandos.

Para iniciar Workerman, primero se necesita un archivo de entrada que defina el puerto y el protocolo que el servicio escuchará. Puede consultar la [sección de ejemplos simples](../getting-started/simple-example.md) para obtener más información.

Tomando como ejemplo [workerman-chat](https://www.workerman.net/workerman-chat), su archivo de entrada de inicio es start.php.

### Iniciar

Iniciar en modo debug

```php start.php start```

Iniciar en modo daemon

```php start.php start -d```

### Detener
```php start.php stop```

### Reiniciar
```php start.php restart```

### Reinicio suave
```php start.php reload```

### Ver estado
```php start.php status```

### Ver estado de conexiones (requiere Workerman versión>=3.5.0)
```php start.php connections```

## Diferencias entre el modo debug y el modo daemon

1. Al iniciar en modo debug, las funciones de impresión como echo, var_dump, print, etc., se mostrarán directamente en la terminal.
2. Al iniciar en modo daemon, las funciones de impresión como echo, var_dump, print, etc., se redirigirán de forma predeterminada al archivo /dev/null, pero se puede configurar el archivo de salida utilizando ```Worker::$stdoutFile = '/your/path/file';```.
3. Al iniciar en modo debug, Workerman se cerrará y finalizará cuando se cierre la terminal.
4. Al iniciar en modo daemon, Workerman continuará ejecutándose en segundo plano después de que se cierre la terminal.

## ¿Qué es un reinicio suave?

Consulte [Principio del reinicio suave](../faq/reload-principle.md)
