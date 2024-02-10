# Проверка отключенных функций

Используйте этот скрипт для проверки отключенных функций. Запустите команду в командной строке: ```curl -Ss https://www.workerman.net/check | php```.

Если вы видите сообщение ```Function название_функции may be disabled. Please check disable_functions in php.ini```, это означает, что функции, от которых зависит workerman, были отключены, и для корректной работы workerman необходимо разрешить их в настройках php.ini.
Для отключения отключенных функций используйте один из следующих методов.

## Метод 1: Разрешение через скрипт

Выполните скрипт `curl -Ss https://www.workerman.net/fix | php` для разрешения отключенных функций.

## Метод 2: Ручное разрешение

**Шаги:**

1. Запустите `php --ini`, чтобы найти расположение файла php.ini, используемого для php cli.
2. Откройте php.ini, найдите параметр `disable_functions` и разрешите отключенные функции.

**Необходимые функции**
Для использования workerman необходимо разрешить следующие функции:
```php
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
