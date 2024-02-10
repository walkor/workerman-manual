# 網路封包擷取

在以下的示例中，我們將透過 ```tcpdump``` 來檢視 ```workerman-chat``` 應用程式透過 ```websocket``` 傳輸的資料。在 ```workerman-chat``` 範例中，伺服器使用 ```7272``` 埠口提供 ```websocket``` 服務，因此我們會擷取在埠口 ```7272``` 上的數據包。

1. *執行指令*：```tcpdump -Ans 4096 -iany port 7272```

2. 在瀏覽器地址欄輸入 ```http://127.0.0.1:55151```

3. 輸入暱稱 ```mynick```

4. 在發表框中輸入 ```hi, all ！```

*最後擷取到的資料如下：*

``` 
/*
 * TCP 三向交握的第一次
 * 瀏覽器本地埠口 60653 向遠端埠口 7272 發送 SYN 封包
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
............

/*
 * TCP 三向交握的第二次
 * 遠端埠口 7272 向瀏覽器埠口 60653 回應 SYN+ACK 封包
 */
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....
............

/*
 * TCP 三向交握的第三次，完成 TCP 連接
 * 瀏覽器本地埠口 60653 向遠端埠口 7272 發送 ACK 封包
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....
............

/*
 * WebSocket 連接的建立
 * 瀏覽器本地埠口 60653 向遠端埠口 7272 發送 WebSocket 連接請求資料
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
```
...（後續資料請參照原始資料）
