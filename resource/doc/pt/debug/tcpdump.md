# Captura de pacotes de rede

No exemplo a seguir, usamos o `tcpdump` para visualizar os dados transmitidos pelo aplicativo `workerman-chat` via `websocket`. No exemplo do `workerman-chat`, o servidor é configurado para fornecer serviços de `websocket` na porta `7272`, então capturamos os pacotes de dados nessa porta.

1. *Executar o comando* `tcpdump -Ans 4096 -iany port 7272`

2. Na barra de endereços do navegador, digite `http://127.0.0.1:55151`

3. Insira o apelido `mynick`

4. No campo de publicação, digite `oi, pessoal !`

*Os dados capturados final resultantes são os seguintes:*

```plaintext
/*
 * Primeiro handshake TCP
 * A porta local 60653 envia um pacote SYN para a porta remota 7272
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
............

/*
 * Segundo handshake TCP
 * A porta remota 7272 responde à porta do navegador 60653 com um pacote SYN+ACK
 */
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....
............

/*
 * Terceiro handshake TCP, conexão TCP concluída
 * A porta local 60653 envia um pacote ACK para a porta remota 7272
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....
........

/*
 * Handshake do websocket
 * A porta local 60653 envia dados de solicitação de handshake do websocket para a porta remota 7272
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
 * Handshake do websocket
 * A porta remota 7272 envia um pacote ACK para a porta do navegador 60653, indicando que recebeu os dados de solicitação de handshake do websocket
 */
17:50:00.524423 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [.], ack 716, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4(u@.@..M.........h..)I....lf.....(.....
........

/*
 * Handshake do websocket
 * A porta remota 7272 envia uma resposta de handshake do websocket, indicando que o handshake foi bem-sucedido
 */
17:50:00.535918 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 1:157, ack 716, win 256, options [nop,nop,TS val 28679669 ecr 28679666], length 156
E...(v@.@............h..)I....lf...........
........HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Sec-WebSocket-Version: 13
Connection: Upgrade
Sec-WebSocket-Accept: nSsCeIBUsFnDJCRb/BNlFzBUDpM=

/*
 * Handshake do websocket bem-sucedido
 * A porta local 60653 envia um ACK, indicando que recebeu a resposta de handshake do websocket
 */
17:50:00.535932 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 157, win 256, options [nop,nop,TS val 28679669 ecr 28679669], length 0
E..4.k@.@.HV...........h..lf)I.#.....(.....
........

/*
 * Pedido de inserção de apelido
 * O navegador envia uma solicitação de inserção de apelido para a porta 7272 via protocolo websocket {"type":"login","name":"mynick"}
*Since the data sent by the browser to the server is masked using the websocket protocol, the original text {"type":"login","name":"mynick"} cannot be seen
 */
17:50:30.652680 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 716:754, ack 157, win 256, options [nop,nop,TS val 28687198 ecr 28679669], length 38
E..Z.l@.@.H/...........h..lf)I.#.....N.....
...^.......&_...+..C}..J0..H}..H>...e.._1..M}.

/*
 * Pedido de inserção de apelido
 * A porta 7272 responde ao navegador com um ACK, indicando que a solicitação de apelido foi recebida e retorna a lista de usuários {"type":"user_list" ...
 */
17:50:30.653546 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 157:267, ack 754, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 110
E...(w@.@............h..)I.#..l............
...^...^.l{"type":"user_list","user_list":[{"uid":783654164,"name":"\u732a\u732a"},{"uid":783700053,"name":"mynick"}]}

/*
 * Pedido de inserção de apelido
 * O navegador responde com um ACK, indicando que os dados da lista de usuários foram recebidos
 */
17:50:30.653559 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 267, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.m@.@.HT...........h..l.)I.......(.....
...^...^

/*
 * Pedido de inserção de apelido
 * A porta 7272 responde com um ACK e retorna o resultado do login {"type":"login",...
 */
17:50:30.653689 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 267:346, ack 754, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 79
E...(x@.@............h..)I....l......w.....
...^...^.M{"type":"login","uid":783700053,"name":"mynick","time":"2014-08-12 17:50:30"}

/*
 * Pedido de inserção de apelido concluído
 * O navegador responde com um ACK, indicando que os dados do resultado do login foram recebidos
 */
17:50:30.653695 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 346, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.n@.@.HS...........h..l.)I.......(.....
...^...^

/*
 * A porta 7272 informa a outros navegadores que há um novo usuário logado
 */
17:50:30.653749 IP 127.0.0.1.7272 > 127.0.0.1.60584: Flags [P.], seq 436:515, ack 816, win 256, options [nop,nop,TS val 28687198 ecr 28577913], length 79
E.....@.@.3..........h..f....G.......w.....
...^...y.M{"type":"login","uid":783700053,"name":"mynick","time":"2014-08-12 17:50:30"}

/*
 * Outros navegadores respondem com ACK, indicando que receberam a notificação de novo usuário logado
 */
17:50:30.653755 IP 127.0.0.1.60584 > 127.0.0.1.7272: Flags [.], ack 515, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.X@.@.#j...........h.G..f..$.....(.....
...^...^

/*
 * Usuário mynick envia mensagem oi, pessoal !
 * O navegador envia dados de mensagem para a porta 7272 via websocket {"type":"say","to_uid":"all","content":"oi, pessoal !"}
*Since the data sent by the browser to the server is masked using the websocket protocol, the original text {"type":"say","to_uid":"all","content":"oi, pessoal !"} cannot be seen
 */
17:51:02.775205 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 754:812, ack 346, win 256, options [nop,nop,TS val 28695228 ecr 28687198], length 58
E..n.o@.@.H............h..l.)I.......b.....
fTX.d.P[(...9H..C=LT.~.BV=...0SnB-X.

/*
 * Usuário mynick envia mensagem oi, pessoal !
 * A porta 7272 envia a um navegador cliente específico os dados da mensagem {"type":"say","from_uid":....
 */
17:51:02.776785 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 346:448, ack 812, win 256, options [nop,nop,TS val 28695229 ecr 28695228], length 102
E...(y@.@............h..)I....l............
.........d{"type":"say","from_uid":783700053,"to_uid":"all","content":"oi, pessoal !","time":"2014-08-12 :51:02"}

/*
 * Usuário mynick envia mensagem oi, pessoal !
 * O navegador responde com ACK, indicando que os dados da mensagem foram recebidos
 */
17:51:02.776808 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 448, win 256, options [nop,nop,TS val 28695229 ecr 28695229], length 0
E..4.p@.@.HQ...........h..l.)I.F.....(.....
........

/*
 * Usuário mynick envia mensagem oi, pessoal !
 * A porta 7272 envia a um navegador cliente específico os dados da mensagem {"type":"say","from_uid":....
 */
17:51:02.776827 IP 127.0.0.1.7272 > 127.0.0.1.60584: Flags [P.], seq 515:617, ack 816, win 256, options [nop,nop,TS val 28695229 ecr 28687198], length 102
E.....@.@.3g.........h..f..$.G.............
.......^.d{"type":"say","from_uid":783700053,"to_uid":"all","content":"oi, pessoal !","time":"2014-08-12 :51:02"}

/*
 * Usuário mynick envia a mensagem oi, pessoal !. Todos os navegadores recebem os dados da mensagem, concluindo o envio da mensagem
 * O navegador responde com ACK, indicando que os dados da mensagem foram recebidos
 */
17:51:02.776842 IP 127.0.0.1.60584 > 127.0.0.1.7272: Flags [.], ack 617, win 256, options [nop,nop,TS val 28695229 ecr 28695229], length 0
E..4.Y@.@.#i...........h.G..f........(.....
........
```


*Isso representa todas as solicitações de login e postagem, com um total de dois clientes de navegador.*

*No pacote de dados, `[S]` representa uma solicitação `[SYN]` (início da solicitação de conexão); `[.]` representa uma resposta `[ACK]`, indicando que a solicitação foi recebida pelo destinatário; `[P]` representa envio de dados; `[P.]` representa `[P] + [.]`.*

*Se os dados transmitidos pela porta forem binários, eles podem ser visualizados em formato hexadecimal: `tcpdump -XAns 4096 -iany port 7272`*
