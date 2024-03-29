# Запуск и остановка

Обратите внимание, что команды для запуска и остановки Workerman выполняются в командной строке.

Для запуска Workerman сначала необходимо создать файл для запуска, в котором определены порт и протокол, на которых будет слушать сервер. Можно ознакомиться с разделом [Пример простого разработчика ввода - раздел базового примера](../getting-started/simple-example.md)

Возьмем, к примеру, [workerman-chat](https://www.workerman.net/workerman-chat), у него файл запуска называется start.php.

### Запуск

Запуск в режиме отладки (debug):

 ```php start.php start```

Запуск в режиме демона (daemon):

 ```php start.php start -d```

### Остановка
 ```php start.php stop```

### Перезапуск
 ```php start.php restart```

### Гладкая перезагрузка
 ```php start.php reload```

### Просмотр статуса
 ```php start.php status```
 
### Просмотр состояния подключений (доступно в версии Workerman >= 3.5.0)
```php start.php connections```



## Разница между режимами debug и daemon

1. При запуске в режиме отладки код с функциями вывода, такими как echo, var_dump, print, будет непосредственно выводиться в терминале.

2. При запуске в режиме демона код с функциями вывода, такими как echo, var_dump, print, будет по умолчанию перенаправляться в файл /dev/null, который можно изменить с помощью установки ```Worker::$stdoutFile = '/your/path/file';``` для указания пути к новому файлу.

3. При запуске в режиме отладки Workerman будет закрыт и завершит работу после закрытия терминала.

4. При запуске в режиме демона Workerman будет продолжать работу в фоновом режиме после закрытия терминала.


## Что такое гладкая перезагрузка?

См. [Принцип гладкой перезагрузки](../faq/reload-principle.md)
