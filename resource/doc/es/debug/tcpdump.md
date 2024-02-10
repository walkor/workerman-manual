# Captura de paquetes de red

En el siguiente ejemplo, usaremos `tcpdump` para ver los datos transmitidos a través de `websocket` en la aplicación `workerman-chat`. En el ejemplo de `workerman-chat`, el servidor ofrece servicios `websocket` en el puerto `7272`, por lo que vamos a capturar los paquetes de datos en el puerto `7272`.

1.  *Ejecuta el comando* `tcpdump -Ans 4096 -iany port 7272`

2. En la barra de direcciones del navegador, ingresa `http://127.0.0.1:55151`

3. Ingresa el apodo `mynick`

4. En el cuadro de publicación, escribe `¡Hola a todos!`

*Los datos capturados finalmente serán:*

```plaintext
/*
 * Handshake TCP inicial
 * El puerto local 60653 del navegador envía paquete SYN al puerto remoto 7272
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
............

/*
 * Segundo paso del Handshake TCP
 * El puerto remoto 7272 responde al puerto del navegador 60653 con paquete SYN+ACK
 */
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....
............

/*
 * Tercer paso del Handshake TCP, conexión TCP completada
 * El puerto local 60653 del navegador envía paquete ACK al puerto remoto 7272
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....
.........

/*
 * Handshake de WebSocket
 * El puerto local 60653 del navegador envía datos de solicitud de Handshake de WebSocket al puerto remoto 7272
 */
17:50:00.524412 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 1:716, ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 715
E....j@.@.E............h..i.)I.............
........GET / HTTP/1.1
Host: 127.0.0.1:7272
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:31.0) Gecko/20100101 Firefox/31.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-cn,zh;q=0.8,en-us;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Sec-WebSocket-Version: 13
Origin: http://127.0.0.1:55151
Sec-WebSocket-Key: zPDr6m4czzUdOFnsxIUEAw==
Cookie: Hm_lvt_abcf9330bef79b4aba5b24fa373506d9=1402048017; Hm_lvt_5fedb3bdce89499492c079ab4a8a0323=1403063068,1403141761; Hm_lvt_7b1919221e89d2aa5711e4deb935debd=1407836536; Hm_lpvt_7b1919221e89d2aa5711e4deb935debd=1407837000
Connection: keep-alive, Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket

/*
 * Handshake de WebSocket
 * El puerto remoto 7272 envía un paquete ACK al puerto del navegador 60653, indicando que ha recibido la solicitud de Handshake de WebSocket
 */
17:50:00.524423 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [.], ack 716, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4(u@.@..M.........h..)I....lf.....(.....
.........

/*
 * Handshake de WebSocket
 * El puerto remoto 7272 envía una respuesta de Handshake de WebSocket al puerto del navegador 60653, indicando que el Handshake se ha completado
 */
17:50:00.535918 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 1:157, ack 716, win 256, options [nop,nop,TS val 28679669 ecr 28679666], length 156
E...(v@.@............h..)I....lf.............
........HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Sec-WebSocket-Version: 13
Connection: Upgrade
Sec-WebSocket-Accept: nSsCeIBUsFnDJCRb/BNlFzBUDpM=

/*
 * Handshake de WebSocket completado
 * El puerto local 60653 del navegador envía un paquete ACK al puerto remoto 7272, indicando que ha recibido la respuesta de Handshake de WebSocket
 */
17:50:00.535932 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 157, win 256, options [nop,nop,TS val 28679669 ecr 28679669], length 0
E..4.k@.@.HV...........h..lf)I.#.....(.....
........

/*
 * Solicitud de ingreso de apodo
 * El navegador envía la solicitud de ingreso de apodo {"type":"login","name":"mynick"} al puerto 7272 mediante WebSocket
 * Los datos enviados por el navegador al servidor están enmascarados por el protocolo WebSocket, por lo que no se puede ver el texto original {"type":"login","name":"mynick"}
 */
17:50:30.652680 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 716:754, ack 157, win 256, options [nop,nop,TS val 28687198 ecr 28679669], length 38
E..Z.l@.@.H/...........h..lf)I.#.....N.....
...^.......&_...+..C}..J0..H}..H>...e.._1..M}.

/*
 * Solicitud de ingreso de apodo
 * El puerto 7272 responde al navegador con un paquete ACK, indicando que la solicitud de ingreso de apodo ha sido recibida, y devuelve la lista de usuarios {"type":"user_list" ...
 */
17:50:30.653546 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 157:267, ack 754, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 110
E...(w@.@............h..)I.#..l............
...^...^.l{"type":"user_list","user_list":[{"uid":783654164,"name":"\u732a\u732a"},{"uid":783700053,"name":"mynick"}]}

/*
 * Solicitud de ingreso de apodo
 * El navegador responde con un paquete ACK, indicando que ha recibido la lista de usuarios
 */
17:50:30.653559 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 267, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.m@.@.HT...........h..l.)I.......(.....
...^...^

/*
 * Solicitud de ingreso de apodo
 * El puerto 7272 responde con un paquete ACK y devuelve el resultado del inicio de sesión {"type":"login",...
 */
17:50:30.653689 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 267:346, ack 754, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 79
E...(x@.@............h..)I....l......w.....
...^...^.M{"type":"login","uid":783700053,"name":"mynick","time":"2014-08-12 17:50:30"}

/*
 * Solicitud de ingreso de apodo completa
 * El navegador responde con un paquete ACK, indicando que ha recibido el paquete de resultado del inicio de sesión
 */
17:50:30.653695 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 346, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.n@.@.HS...........h..l.)I.......(.....
...^...^

/*
 * El puerto 7272 notifica a los demás navegadores que hay un nuevo usuario conectado
 */
17:50:30.653749 IP 127.0.0.1.7272 > 127.0.0.1.60584: Flags [P.], seq 436:515, ack 816, win 256, options [nop,nop,TS val 28687198 ecr 28577913], length 79
E.....@.@.3..........h..f....G.......w.....
...^...y.M{"type":"login","uid":783700053,"name":"mynick","time":"2014-08-12 17:50:30"}

/*
 * El otro navegador responde con ACK, indicando que ha recibido la notificación de nuevo inicio de sesión
 */
17:50:30.653755 IP 127.0.0.1.60584 > 127.0.0.1.7272: Flags [.], ack 515, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.X@.@.#j...........h.G..f..$.....(.....
...^...^

/*
 * mynick envía el mensaje ¡Hola a todos!
 * El navegador envía datos de mensaje {"type":"say","to_uid":"all","content":"hi,  all !"} al puerto 7272 a través de WebSocket
 * Los datos enviados por el navegador al servidor están enmascarados por el protocolo WebSocket, por lo que no se puede ver el texto original {"type":"say","to_uid":"all","content":"hi,  all !"}
 */
17:51:02.775205 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 754:812, ack 346, win 256, options [nop,nop,TS val 28695228 ecr 28687198], length 58
E..n.o@.@.H............h..l.)I.......b.....
fTX.d.P[(...9H..C=LT.~.BV=...0SnB-X.

/*
 * mynick envía el mensaje ¡Hola a todos!
 * El puerto 7272 reenvía los datos del mensaje a un navegador cliente entre todos los navegadores clientes mediante paquete JSON {"type":"say","from_uid":....
 */
17:51:02.776785 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 346:448, ack 812, win 256, options [nop,nop,TS val 28695229 ecr 28695228], length 102
E...(y@.@............h..)I....l............
.........d{"type":"say","from_uid":783700053,"to_uid":"all","content":"hi,  all !","time":"2014-08-12 :51:02"}

/*
 * mynick envía el mensaje ¡Hola a todos!
 * El navegador responde con ACK, indicando que ha recibido los datos del mensaje
 */
17:51:02.776808 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 448, win 256, options [nop,nop,TS val 28695229 ecr 28695229], length 0
E..4.p@.@.HQ...........h..l.)I.F.....(.....
........

/*
 * mynick envía el mensaje ¡Hola a todos!
 * El puerto 7272 reenvía los datos del mensaje a un navegador cliente entre todos los navegadores clientes mediante paquete JSON {"type":"say","from_uid":....
 */
17:51:02.776827 IP 127.0.0.1.7272 > 127.0.0.1.60584: Flags [P.], seq 515:617, ack 816, win 256, options [nop,nop,TS val 28695229 ecr 28687198], length 102
E.....@.@.3g.........h..f..$.G.............
.......^.d{"type":"say","from_uid":783700053,"to_uid":"all","content":"hi,  all !","time":"2014-08-12 :51:02"}

/*
 * mynick envía el mensaje ¡Hola a todos! Todos los navegadores reciben los datos del mensaje y termina la transmisión del mensaje
 * El navegador responde con ACK, indicando que ha recibido los datos del mensaje
 */
17:51:02.776842 IP 127.0.0.1.60584 > 127.0.0.1.7272: Flags [.], ack 617, win 256, options [nop,nop,TS val 28695229 ecr 28695229], length 0
E..4.Y@.@.#i...........h.G..f........(.....
........
``` 

*Estas son todas las solicitudes de inicio de sesión y mensajes enviados, un total de dos clientes de navegador.* 

*En los datos de los paquetes, `[S]` representa una solicitud `SYN` (iniciar la conexión); `[.]` representa una respuesta `ACK`, indicando que el extremo receptor ha recibido la solicitud; `[P]` representa el envío de datos; `[P.]` representa `[P] + [.]*

*Si los datos transmitidos por el puerto son datos binarios, se pueden ver en hexadecimal con el comando* `tcpdump -XAns 4096 -iany port 7272`
