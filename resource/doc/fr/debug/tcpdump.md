# Capture réseau

Dans l'exemple suivant, nous utilisons `tcpdump` pour visualiser les données transmises par l'application `workerman-chat` via WebSocket. Dans l'exemple de `workerman-chat`, le serveur fournit des services WebSocket sur le port `7272`, nous capturons donc les paquets de données sur le port `7272`.

1. *Exécuter la commande* `tcpdump -Ans 4096 -iany port 7272`

2. Dans la barre d'adresse du navigateur, entrez `http://127.0.0.1:55151`

3. Entrez le surnom `mynick`

4. Dans la zone de publication, saisissez `hi, all !`

*Les données capturées finales sont les suivantes :*

```plaintext
/*
 * Première poignée de main TCP
 * Le port local 60653 envoie le paquet SYN vers le port distant 7272
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
............

/*
 * Deuxième poignée de main TCP
 * Le port distant 7272 répond au port local 60653 avec un paquet SYN+ACK
 */
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....
............

/*
 * Troisième poignée de main TCP, connexion TCP établie
 * Le port local 60653 envoie un paquet ACK vers le port distant 7272
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....
........

/*
 * Poignée de main WebSocket
 * Le port local 60653 envoie les données de requête de poignée de main WebSocket vers le port distant 7272
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

// ... (suite des données capturées)

```

*Cela représente toutes les requêtes de connexion et de publication. Il y a au total deux clients navigateur.*

*Dans les données de paquet, `[S]` indique une requête `SYN` (initiation de la connexion) ; `[.]` indique une réponse `ACK`, confirmant la réception de la demande par l'autre partie ; `[P]` indique l'envoi de données ; `[P.]` indique `[P]` + `[.]`*

*Si les données transmises via le port sont en format binaire, elles peuvent être visualisées en format hexadécimal :* `tcpdump -XAns 4096 -iany port 7272`
