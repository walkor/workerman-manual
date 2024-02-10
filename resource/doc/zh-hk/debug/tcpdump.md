# 網絡抓包

在以下例子中，我們使用```tcpdump```查看```workerman-chat```應用透過```websocket```傳輸的資料。```workerman-chat```的服務端使用```7272```端口來提供```websocket```服務，所以我們抓取```7272```端口上的數據包。

1、*執行命令* ```tcpdump -Ans 4096 -iany port 7272```

2、在瀏覽器地址欄輸入```http://127.0.0.1:55151```

3、輸入暱稱```mynick```

4、在發表框輸入```hi, all ！```

*最終抓取到的數據如下：*

```
/* 
 * TCP 三次握手
 * 本機端口60653向遠程端口7272發送SYN包
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
............

/* 
 * TCP 第二次握手
 * 遠程端口7272向本機端口60653回應SYN+ACK包
 */
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....
............

/* 
 * TCP 第三次握手，完成TCP連接
 * 本機端口60653向遠程端口7272發送ACK包
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....
............

/* 
 * websocket握手
 * 本機端口60653向遠程端口7272發送websocket握手請求數據
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
 * websocket 握手
 * 遠程端口7272向本機端口60653發送ACK包，表示遠程7272端口已經收到websocket握手請求數據
 */
17:50:00.524423 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [.], ack 716, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4(u@.@..M.........h..)I....lf.....(.....
............

/* 
 * websocket 握手
 * 遠程端口7272向本機端口60653發送websocket握手回應，表示握手成功
 */
17:50:00.535918 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 1:157, ack 716, win 256, options [nop,nop,TS val 28679669 ecr 28679666], length 156
E...(v@.@............h..)I....lf...........
........HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Sec-WebSocket-Version: 13
Connection: Upgrade
Sec-WebSocket-Accept: nSsCeIBUsFnDJCRb/BNlFzBUDpM=

/* 
 * websocket 握手成功
 * 本機端口60653向遠程端口7272發送ACK，表示接收到websocket握手回應數據
 */
17:50:00.535932 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 157, win 256, options [nop,nop,TS val 28679669 ecr 28679669], length 0
E..4.k@.@.HV...........h..lf)I.#.....(.....
............

/* 
 * 輸入暱稱請求
 * 經由websocket協議，瀏覽器向7272端口發送 暱稱 請求 {"type":"login","name":"mynick"}
 * 由於瀏覽器向服務端發送的數據為websocket協議掩碼處理過的數據，所以無法看到原文 {"type":"login","name":"mynick"}
 */
17:50:30.652680 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 716:754, ack 157, win 256, options [nop,nop,TS val 28687198 ecr 28679669], length 38
E..Z.l@.@.H/...........h..lf)I.#.....N.....
...^.......&_...+..C}..J0..H}..H>...e.._1..M}.

/* 
 * 輸入暱稱請求
 * 7272端口向瀏覽器返回ACK，表示暱稱請求已經接收，並返回用戶列表{"type":"user_list" ...
 */
17:50:30.653546 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 157:267, ack 754, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 110
E...(w@.@............h..)I.#..l.............
...^...^.l{"type":"user_list","user_list":[{"uid":783654164,"name":"\u732a\u732a"},{"uid":783700053,"name":"mynick"}]}

/* 
 * 輸入暱稱請求
 * 瀏覽器返回ACK，表示用戶列表數據已經收到
 */
17:50:30.653559 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 267, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.m@.@.HT...........h..l.)I.......(.....
...^...^

/* 
 * 輸入暱稱請求
 * 7272端口向瀏覽器返回ACK，並返回用戶登錄結果{"type":"login",...
 */
17:50:30.653689 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 267:346, ack 754, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 79
E...(x@.@............h..)I....l......w.....
...^...^.M{"type":"login","uid":783700053,"name":"mynick","time":"2014-08-12 17:50:30"}

/* 
 * 輸入暱稱請求 完畢
 * 瀏覽器返回ACK，表示登錄結果數據包收到
 */
17:50:30.653695 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 346, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.n@.@.HS...........h..l.)I.......(.....
...^...^

/* 
 * 服務端7272端口通知其他瀏覽器有新用戶登錄
 */
17:50:30.653749 IP 127.0.0.1.7272 > 127.0.0.1.60584: Flags [P.], seq 436:515, ack 816, win 256, options [nop,nop,TS val 28687198 ecr 28577913], length 79
E.....@.@.3..........h..f....G.......w.....
...^...y.M{"type":"login","uid":783700053,"name":"mynick","time":"2014-08-12 17:50:30"}

/* 
 * 其他瀏覽器返回ACK，表示收到新用戶登錄通知的請求
 */
17:50:30.653755 IP 127.0.0.1.60584 > 127.0.0.1.7272: Flags [.], ack 515, win 256, options [nop,nop,TS val 28687198 ecr 28687198], length 0
E..4.X@.@.#j...........h.G..f..$.....(.....
...^...^

/* 
 * mynick用戶發言 hi, all !
 * 瀏覽器向服務端7272端口發送發言數據 {"type":"say","to_uid":"all","content":"hi,  all !"}
 * 由於瀏覽器向服務端發送的數據為websocket協議掩碼處理過的數據，所以無法看到原文
 */
17:51:02.775205 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 754:812, ack 346, win 256, options [nop,nop,TS val 28695228 ecr 28687198], length 58
E..n.o@.@.H............h..l.)I.......b.....
fTX.d.P[(...9H..C=LT.~.BV=...0SnB-X.

/* 
 * mynick用戶發言 hi, all !
 * 7272端口向所有瀏覽器客戶端中一個瀏覽器轉發發言數據 {"type":"say","from_uid":....
 */
17:51:02.776785 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [P.], seq 346:448, ack 812, win 256, options [nop,nop,TS val 28695229 ecr 28695228], length 102
E...(y@.@............h..)I....l.............
.........d{"type":"say","from_uid":783700053,"to_uid":"all","content":"hi,  all !","time":"2014-08-12 :51:02"}

/* 
 * mynick用戶發言 hi, all !
 * 瀏覽器響應ACK，收到發言數據
 */
17:51:02.776808 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 448, win 256, options [nop,nop,TS val 28695229 ecr 28695229], length 0
E..4.p@.@.HQ...........h..l.)I.F.....(.....
............

/* 
 * mynick用戶發言 hi, all !
 * 7272端口向所有瀏覽器客戶端中一個瀏覽器轉發發言數據 {"type":"say","from_uid":....
 */
17:51:02.776827 IP 127.0.0.1.7272 > 127.0.0.1.60584: Flags [P.], seq 515:617, ack 816, win 256, options [nop,nop,TS val 28695229 ecr 28687198], length 102
E.....@.@.3g.........h..f..$.G.............
.......^.d{"type":"say","from_uid":783700053,"to_uid":"all","content":"hi,  all !","time":"2014-08-12 :51:02"}

/* 
 * mynick用戶發言 hi, all ! ，所有瀏覽器都收到轉發的發言數據，發言完畢
 * 瀏覽器響應ACK，收到發言數據
 */
17:51:02.776842 IP 127.0.0.1.60584 > 127.0.0.1.7272: Flags [.], ack 617, win 256, options [nop,nop,TS val 28695229 ecr 28695229], length 0
E..4.Y@.@.#i...........h.G..f........(.....
............

``` 

*以上是登錄+發言的所有所有請求，總共有兩個瀏覽器客戶端。*

*數據包中的```[S]```代表```SYN```請求（發起連接請求）；```[.]```代表```ACK```回應，說明請求對端已經收到；[```P```]代表發送數據；[P.]代表[P] + [.]*

*如果端口上傳輸的數據是二進制數據，則可以以十六進制來查看* ```tcpdump -XAns 4096 -iany port 7272```
