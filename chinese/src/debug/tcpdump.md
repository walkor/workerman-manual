# 网络抓包

下面的例子中我们通过```tcpdump```查看```workerman-chat```应用通过```websocket```传输的数据。```workerman-chat```例子中服务端是通过```7272```端口对外提供```websocket```服务的，所以我们抓取```7272```端口上的数据包。

1、*运行命令* ```tcpdump -Ans 4096 -iany port 7272```

2、在浏览器地址栏输入 ```http://127.0.0.1:55151```

3、输入昵称 ```mynick```

4、发表框输入 ```hi, all ！```

*最终抓取的数据如下:*

```
/*
 * TCP第一次握手
 * 浏览器本地端口60653向远程端口7272发送SYN包
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
............

/*
 * TCP第二次握手
 * 远程端口7272向浏览器端口60653回应SYN+ACK包
 */
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....
............

/*
 * TCP第三次握手，完成TCP链接
 * 浏览器本地端口60653向远程端口7272发送ACK包
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....
........

/*
 * websocket握手
 * 浏览器本地端口60653向远程端口7272发送websocket握手请求数据
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
 * websocket握手
 * 远程端口7272向浏览器端口60653发送ACK包，表明远程7272端口已经收到websocket握手请求数据
 */
17:50:00.524423 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [.], ack 716, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4(u@.@..M.........h..)I....lf.....(.....
........

/*
 * websocket握手
 * 远程端口7272向浏览器端口60653发送websocket握手回应，表明握手成功
 */
17:50:00.535918 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 1:157, ack 716, win 256, options [nop,nop,TS val 28679669 ecr 28679666], length 156
E...(v@.@............h..)I....lf...........
........HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Sec-WebSocket-Version: 13
Connection: Upgrade
Sec-WebSocket-Accept: nSsCeIBUsFnDJCRb/BNlFzBUDpM=

/*
 * websocket握手成功
 * 浏览器本地端口60653向远程端口7272发送ACK，表明接收到websocket握手回应数据
 */
17:50:00.535932 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 157, win 256, options [nop,nop,TS val 28679669 ecr 28679669], length 0
E..4.k@.@.HV...........h..lf)I.#.....(.....
........

/*
 * 输入昵称请求
 * 浏览器通过websocket协议向7272端口发送 昵称 请求 {"type":"login","name":"mynick"}
 * 由于浏览器向服务端发送的数据为websocket协议掩码处理过的数据，所以无法看到原文 {"type":"login","name":"mynick"}
 */
17:50:30.652680 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 716:754, ack 157, win 256, options [nop,nop,TS val 28687198 ecr 28679669], length 38
E..Z.l@.@.H/...........h..lf)I.#.....N.....
...^.......&_...+..C}..J0..H}..H>...e.._1..M}.

/*
 * 输入昵称请求
 * 7272端口向浏览器返回ACK，表明昵称请求已经接收，并返回用户列表{"type":"user_list" ...
 */
17:50:30.653546 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 157:267, ack 754, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 110
E...(w@.@............h..)I.#..l............
...^...^.l{"type":"user_list","user_list":[{"uid":783654164,"name":"\u732a\u732a"},{"uid":783700053,"name":"mynick"}]}

/*
 * 输入昵称请求
 * 浏览器返回ACK，表明用户列表数据已经收到
 */
17:50:30.653559 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 267, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.m@.@.HT...........h..l.)I.......(.....
...^...^

/*
 * 输入昵称请求
 * 7272端口向浏览器返回ACK，并返回用登录结果{"type":"login",...
 */
17:50:30.653689 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 267:346, ack 754, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 79
E...(x@.@............h..)I....l......w.....
...^...^.M{"type":"login","uid":783700053,"name":"mynick","time":"2014-08-12 17:50:30"}

/*
 * 输入昵称请求 完毕
 * 浏览器返回ACK，表明登录结果数据包收到
 */
17:50:30.653695 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 346, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.n@.@.HS...........h..l.)I.......(.....
...^...^

/*
 * 服务端7272端口通知其它浏览器有新用户登录
 */
17:50:30.653749 IP 127.0.0.1.7272 > 127.0.0.1.60584: Flags [P.], seq 436:515, ack 816, win 256, options [nop,nop,TS val 28687198 ecr 28577913], length 79
E.....@.@.3..........h..f....G.......w.....
...^...y.M{"type":"login","uid":783700053,"name":"mynick","time":"2014-08-12 17:50:30"}

/*
 * 其它浏览器返回 ACK,表明收到新用户登录通知的请求
 */
17:50:30.653755 IP 127.0.0.1.60584 > 127.0.0.1.7272: Flags [.], ack 515, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.X@.@.#j...........h.G..f..$.....(.....
...^...^

/*
 * mynick用户发言 hi, all !
 * 浏览器向服务端7272端口发送发言数据 {"type":"say","to_uid":"all","content":"hi,  all !"}
 * 由于浏览器向服务端发送的数据为websocket协议掩码处理过的数据，所以无法看到原文
 */
17:51:02.775205 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 754:812, ack 346, win 256, options [nop,nop,TS val 28695228 ecr 28687198], length 58
E..n.o@.@.H............h..l.)I.......b.....
fTX.d.P[(...9H..C=LT.~.BV=...0SnB-X.

/*
 * mynick用户发言 hi, all !
 * 7272端口向所有浏览器客户端中一个浏览器转发发言数据 {"type":"say","from_uid":....
 */
17:51:02.776785 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 346:448, ack 812, win 256, options [nop,nop,TS val 28695229 ecr 28695228], length 102
E...(y@.@............h..)I....l............
.........d{"type":"say","from_uid":783700053,"to_uid":"all","content":"hi,  all !","time":"2014-08-12 :51:02"}

/*
 * mynick用户发言 hi, all !
 * 浏览器响应ACK，收到发言数据
 */
17:51:02.776808 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 448, win 256, options [nop,nop,TS val 28695229 ecr 28695229], length 0
E..4.p@.@.HQ...........h..l.)I.F.....(.....
........

/*
 * mynick用户发言 hi, all !
 * 7272端口向所有浏览器客户端中一个浏览器转发发言数据 {"type":"say","from_uid":....
 */
17:51:02.776827 IP 127.0.0.1.7272 > 127.0.0.1.60584: Flags [P.], seq 515:617, ack 816, win 256, options [nop,nop,TS val 28695229 ecr 28687198], length 102
E.....@.@.3g.........h..f..$.G.............
.......^.d{"type":"say","from_uid":783700053,"to_uid":"all","content":"hi,  all !","time":"2014-08-12 :51:02"}

/*
 * mynick用户发言 hi, all ! ，所有浏览器都收到转发的发言数据，发言完毕
 * 浏览器响应ACK，收到发言数据
 */
17:51:02.776842 IP 127.0.0.1.60584 > 127.0.0.1.7272: Flags [.], ack 617, win 256, options [nop,nop,TS val 28695229 ecr 28695229], length 0
E..4.Y@.@.#i...........h.G..f........(.....
........


```

*以上是登录+发言的所有所有请求，一共有两个浏览器客户端。*


*包数据中```[S]```代表```SYN```请求（发起链接请求）；```[.]```代表```ACK```回应，说明请求对端已经收到；[```P```]代表发送数据；[P.]代表[P] + [.]*


*如果端口上传输的数据是二进制数据，则可以以十六进制来查看* ```tcpdump -XAns 4096 -iany port 7272```


