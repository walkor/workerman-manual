# Перехват сетевого трафика

В следующем примере мы будем использовать `tcpdump`, чтобы посмотреть данные, передаваемые через протокол `websocket` в приложении `workerman-chat`. В данном примере сервер в приложении `workerman-chat` предоставляет `websocket` сервис через порт `7272`, поэтому мы захватываем пакеты данных на порту `7272`.

1. *Запустите команду* `tcpdump -Ans 4096 -iany port 7272`

2. В адресной строке браузера введите `http://127.0.0.1:55151`

3. Введите никнейм `mynick`

4. Введите сообщение `hi, all !`

*Захваченные данные:*

```
/*
 * Установка TCP-соединения, первый шаг
 * Локальный порт 60653 браузера отправляет пакет SYN на удаленный порт 7272
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....

...

/*
 * Установка TCP-соединения, второй шаг
 * Удаленный порт 7272 отправляет ответный пакет SYN+ACK на локальный порт 60653 браузера
 */ 
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....

...

/*
 * Установка TCP-соединения, третий шаг, установка соединения завершена
 * Локальный порт 60653 браузера отправляет пакет ACK на удаленный порт 7272
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....

...

/*
 * Установка соединения через websocket
 * Локальный порт 60653 браузера отправляет данные запроса на установление соединения через websocket на удаленный порт 7272
 */
17:50:00.524412 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 1:716, ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 715
E....j@.@.E............h..i.)I.............

...

```

*Все запросы на вход и исходные данные сообщений указаны выше. Общее количество запросов составляет два браузерных клиента.*

*В данных пакетов, `[S]` означает запрос `[SYN]` (запрос на установление соединения); `[.]` означает ответ `[ACK]`, указывающий на то, что запрошенный конечный узел уже получил; `[P]` означает отправку данных; `[P.]` означает `[P]` + `[.]`*

*Если передаваемые через порт данные являются двоичными, то их можно просмотреть в шестнадцатеричном формате, используя команду `tcpdump -XAns 4096 -iany port 7272`*
