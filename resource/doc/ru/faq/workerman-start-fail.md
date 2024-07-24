# Не удалось запустить workerman

## Симптом 1
После запуска выдается ошибка, подобная следующей:
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx
```
**Ключевые слова**: ```Address already in use```

**Основная причина**: Порт уже занят и не удается запустить.

#### Решение 1

Можно найти программу, которая заняла порт, выполнив команду ```netstat -anp | grep номер_порта``` и затем остановить соответствующую программу, чтобы освободить порт.

#### Решение 2
Если не удается остановить программу, занимающую порт, можно попробовать изменить порт для Workerman.

#### Решение 3
Если порт занят Workerman'ом, но невозможно остановить его с помощью команды stop (обычно это происходит из-за потери файла PID или убийства главного процесса разработчиком), можно завершить процесс Workerman, выполнив следующие две команды:

```
killall php
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

#### Решение 4
Если действительно нет программы, прослушивающей этот порт, возможно, разработчик задал более одного прослушивания в Workerman с одинаковыми портами. Разработчик должен самостоятельно проверить скрипт запуска на предмет прослушивания одинаковых портов.

#### Решение 5
Проверьте, включена ли опция reusePort; попробуйте отключить ее.

## Симптом 2
После запуска выдается ошибка, подобная следующей:
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
или
```
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (в его контексте запрошенный адрес недействителен) in ...workerman/Worker.php on line xxxx
```
**Ключевые слова**: `Cannot assign requested address` или `в его контексте запрошенный адрес недействителен`

**Причина сбоя**: Неправильно указан ip, который не является локальным; для решения проблемы следует указать локальный ip или ```0.0.0.0``` (чтобы прослушивать все ip локальной машины).

**Совет**: Для Linux можно использовать команду ```ifconfig```, чтобы просмотреть все ip сетевых интерфейсов. Если вы используете облачный сервер (например, Alibaba Cloud/Tencent Cloud и т. д.), обратите внимание, что ваш публичный ip-адрес фактически может быть прокси-адресом (например, специальная сеть Alibaba Cloud), который фактически не является ip-адресом текущего сервера, поэтому его нельзя использовать для прослушивания. Но по-прежнему можно использовать 0.0.0.0 для привязки.

## Симптом 3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**Причина сбоя**: Функция stream_socket_server отключена в php.ini.

**Решение**

1. Запустите ```php --ini```, чтобы найти файл php.ini.
2. Откройте php.ini и удалите отключенную функцию stream_socket_server из списка disable_functions.

## Симптом 4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**Причина сбоя**: На Linux необходимы права root для прослушивания порта меньше 1024.

**Решение**

Используйте порт больше 1024 или запустите службу от имени root пользователя.