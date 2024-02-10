# Network Packet Capturing

In the following example, we use `tcpdump` to view the data transmitted via `websocket` in the `workerman-chat` application. In the `workerman-chat` example, the server provides `websocket` service on port `7272`, so we capture data packets on port `7272`.

1. *Run the command* `tcpdump -Ans 4096 -iany port 7272`

2. Enter `http://127.0.0.1:55151` in the browser address bar

3. Input the nickname `mynick`

4. Enter `hi, all ！` in the message input box

*The captured data is as follows:*

```
/*
 * TCP three-way handshake
 * Local port 60653 sends a SYN packet to remote port 7272
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
............

/*
 * TCP three-way handshake
 * Remote port 7272 responds to local port 60653 with a SYN+ACK packet
 */
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....
............

/*
 * TCP three-way handshake, completing the TCP connection
 * Local port 60653 sends an ACK packet to remote port 7272
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....
........

/*
 * WebSocket handshake
 * Local port 60653 sends websocket handshake request data to remote port 7272
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
 * WebSocket handshake
 * Remote port 7272 sends an ACK packet to local port 60653, indicating that the remote port 7272 has received the websocket handshake request data
 */
17:50:00.524423 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [.], ack 716, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4(u@.@..M.........h..)I....lf.....(.....
........

/*
 * WebSocket handshake
 * Remote port 7272 sends a websocket handshake response to local port 60653, indicating that the handshake is successful
 */
17:50:00.535918 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 1:157, ack 716, win 256, options [nop,nop,TS val 28679669 ecr 28679666], length 156
E...(v@.@............h..)I....lf...........
........HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Sec-WebSocket-Version: 13
Connection: Upgrade
Sec-WebSocket-Accept: nSsCeIBUsFnDJCRb/BNlFzBUDpM=

/*
 * WebSocket handshake successful
 * Local port 60653 sends an ACK to remote port 7272, indicating the reception of the websocket handshake response data
 */
17:50:00.535932 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 157, win 256, options [nop,nop,TS val 28679669 ecr 28679669], length 0
E..4.k@.@.HV...........h..lf)I.#.....(.....
.........

/*
 * Nickname input request
 * Browser sends a nickname request to port 7272 through the websocket protocol {"type":"login","name":"mynick"}
 * Since the data sent from the browser to the server is masked by the websocket protocol, the original text {"type":"login","name":"mynick"} cannot be seen
 */
17:50:30.652680 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 716:754, ack 157, win 256, options [nop,nop,TS val 28687198 ecr 28679669], length 38
E..Z.l@.@.H/...........h..lf)I.#.....N.....
...^.......&_...+..C}..J0..H}..H>...e.._1..M}.

/*
 * Nickname input request
 * Port 7272 acknowledges the nickname request received and returns the user list {"type":"user_list" ...
 */
17:50:30.653546 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 157:267, ack 754, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 110
E...(w@.@............h..)I.#..l.............
...^...^.l{"type":"user_list","user_list":[{"uid":783654164,"name":"\u732a\u732a"},{"uid":783700053,"name":"mynick"}]}

/*
 * Nickname input request
 * Browser acknowledges the user list data received
 */
17:50:30.653559 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 267, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.m@.@.HT...........h..l.)I.......(.....
...^...^

/*
 * Nickname input request
 * Port 7272 acknowledges the user list data received and returns the login result {"type":"login",...
 */
17:50:30.653689 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 267:346, ack 754, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 79
E...(x@.@............h..)I....l......w.....
...^...^.M{"type":"login","uid":783700053,"name":"mynick","time":"2014-08-12 17:50:30"}

/*
 * Nickname input request completed
 * Browser acknowledges the reception of the login result data packet
 */
17:50:30.653695 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 346, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.n@.@.HS...........h..l.)I.......(.....
...^...^

/*
 * Port 7272 notifies other browsers that a new user has logged in
 */
17:50:30.653749 IP 127.0.0.1.7272 > 127.0.0.1.60584: Flags [P.], seq 436:515, ack 816, win 256, options [nop,nop,TS val 28687198 ecr 28577913], length 79
E.....@.@.3..........h..f....G.......w.....
...^...y.M{"type":"login","uid":783700053,"name":"mynick","time":"2014-08-12 17:50:30"}

/*
 * Another browser acknowledges the reception of the notification of the new user login
 */
17:50:30.653755 IP 127.0.0.1.60584 > 127.0.0.1.7272: Flags [.], ack 515, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.X@.@.#j...........h.G..f..$.....(.....
...^...^

/*
 * User `mynick` speaks "hi, all !"
 * Browser sends the speaking data {"type":"say","to_uid":"all","content":"hi,  all !"} to port 7272 using the websocket protocol
 * Since the data sent from the browser to the server is masked by the websocket protocol, the original text {"type":"say","to_uid":"all","content":"hi,  all !"} cannot be seen
 */
17:51:02.775205 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 754:812, ack 346, win 256, options [nop,nop,TS val 28695228 ecr 28687198], length 58
E..n.o@.@.H............h..l.)I.......b.....
fTX.d.P[(...9H..C=LT.~.BV=...0SnB-X.

/*
 * User `mynick` speaks "hi, all !"
 * Port 7272 forwards the speaking data {"type":"say","from_uid":.... to another browser client
 */
17:51:02.776785 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 346:448, ack 812, win 256, options [nop,nop,TS val 28695229 ecr 28695228], length 102
E...(y@.@............h..)I....l.............
.........d{"type":"say","from_uid":783700053,"to_uid":"all","content":"hi,  all !","time":"2014-08-12 :51:02"}

/*
 * User `mynick` speaks "hi, all !"
 * Browser acknowledges the reception of the speaking data
 */
17:51:02.776808 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 448, win 256, options [nop,nop,TS val 28695229 ecr 28695229], length 0
E..4.p@.@.HQ...........h..l.)I.F.....(.....
........

/*
 * User `mynick` speaks "hi, all !"
 * Port 7272 forwards the speaking data {"type":"say","from_uid":.... to another browser client
 */
17:51:02.776827 IP 127.0.0.1.7272 > 127.0.0.1.60584: Flags [P.], seq 515:617, ack 816, win 256, options [nop,nop,TS val 28695229 ecr 28687198], length 102
E.....@.@.3g.........h..f..$.G.............
.......^.d{"type":"say","from_uid":783700053,"to_uid":"all","content":"hi,  all !","time":"2014-08-12 :51:02"}

/*
 * User `mynick` speaks "hi, all !", all browsers receive the forwarded speaking data, and the speaking is completed
 * Browser acknowledges the reception of the speaking data
 */
17:51:02.776842 IP 127.0.0.1.60584 > 127.0.0.1.7272: Flags [.], ack 617, win 256, options [nop,nop,TS val 28695229 ecr 28695229], length 0
E..4.Y@.@.#i...........h.G..f........(.....
........
```

*The above includes all the requests for login and speaking, involving two browser clients.*

*In the packet data, `[S]` represents a `SYN` request (initiating a connection request); `[.]` represents an `ACK` response, indicating that the request has been received by the remote end; `[P]` represents data being sent; `[P.]` represents `[P]` + `[.]`.*

*If the data being transmitted on the port is in binary, it can be viewed in hex using `tcpdump -XAns 4096 -iany port 7272`.*
