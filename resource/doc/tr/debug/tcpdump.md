# Ağ Yakalama

Aşağıdaki örnekte, ```tcpdump``` aracılığıyla ```workerman-chat``` uygulaması tarafından websocket üzerinden iletilen verileri görüntüleyeceğiz. ```workerman-chat``` örneğinde, sunucu ```7272``` portunda websocket hizmeti sunmaktadır, bu nedenle ```7272``` portunda iletilen veri paketlerini yakalayacağız.

1. *Komutu çalıştırın:* ```tcpdump -Ans 4096 -iany port 7272```

2. Tarayıcı adres çubuğuna gidin: ```http://127.0.0.1:55151```

3. Takma adı girin: ```mynick```

4. Mesaj kutusuna yazın: ```Merhaba, herkes!```

*Sonunda yakalanan veriler şu şekildedir:*

```
/*
 * TCP el sıkışması - İlk aşama
 * Tarayıcı yerel portu 60653, uzak port 7272'ye SYN paketi gönderiyor
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
............

/* 
 * TCP el sıkışması - İkinci aşama
 * Uzak port 7272, tarayıcı portu 60653'e SYN+ACK paketi ile yanıt veriyor
 */
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....
............

/*
 * TCP el sıkışması - Üçüncü aşama, TCP bağlantısı tamamlandı
 * Tarayıcı yerel portu 60653, uzak port 7272'ye ACK paketi gönderiyor
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....
........

/*
 * Websocket el sıkışması
 * Tarayıcı yerel portu 60653, uzak port 7272'ye websocket el sıkışma isteği verisi gönderiyor
 */
17:50:00.524412 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 1:716, ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 715
E....j@.@.E............h..i.)I.............
.......GET / HTTP/1.1
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

// ... Diğer adımlar devam eder ...

```

*Yukarıda, giriş ve mesajlaşma isteklerinin tümü ve iki tarayıcı istemcisi olduğu tüm istekler yer almaktadır.*

*Paket verilerinde, ```[S]``` SYN isteğini (bağlantı isteği başlatma) ; ```[.]``` ACK yanıtını, isteğin karşı taraf tarafından alındığını gösterir ; [```P```] ise veri gönderir; [P.] [P] + [.]*

*Eğer port üzerinden iletilen veri ikili ise, heksteki olarak görüntülemek için aşağıdaki komut kullanılır:* ```tcpdump -XAns 4096 -iany port 7272```
