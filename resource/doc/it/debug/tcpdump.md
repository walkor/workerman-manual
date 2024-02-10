# Network Packet Capture

Nell'esempio seguente, utilizziamo `tcpdump` per visualizzare i dati trasmessi tramite `websocket` dall'applicazione `workerman-chat`. Nell'esempio di `workerman-chat`, il server fornisce il servizio `websocket` sulla porta `7272`, quindi catturiamo i pacchetti di dati sulla porta `7272`.

1. *Esecuzione del comando* `tcpdump -Ans 4096 -iany port 7272`

2. Nella barra degli indirizzi del browser, digitare `http://127.0.0.1:55151`

3. Inserire il nickname `mynick`

4. Inserire nel box del messaggio `hi, all ！`

*Di seguito sono riportati i dati catturati:*

```plaintext
/*
 * Handshake TCP iniziale
 * La porta locale 60653 del browser invia il pacchetto SYN alla porta remota 7272
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
............

/*
 * Secondo Handshake TCP
 * La porta remota 7272 risponde con SYN+ACK alla porta locale 60653 del browser
 */
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....
............

/*
 * Terzo Handshake TCP, completamento della connessione TCP
 * La porta locale 60653 del browser invia il pacchetto ACK alla porta remota 7272
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....
........

/*
 * Handshake WebSocket
 * La porta locale 60653 del browser invia i dati di richiesta di Handshake WebSocket alla porta remota 7272
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

...
```
