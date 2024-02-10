# Gói dữ liệu mạng

Trong ví dụ sau, chúng ta sử dụng `tcpdump` để xem dữ liệu được truyền qua `websocket` của ứng dụng `workerman-chat`. Trong ví dụ `workerman-chat`, máy chủ được cung cấp dịch vụ `websocket` qua cổng `7272`, vì vậy chúng ta sẽ bắt gói dữ liệu trên cổng `7272`.

1. *Chạy lệnh* `tcpdump -Ans 4096 -iany port 7272`

2. Trong thanh địa chỉ trình duyệt, nhập `http://127.0.0.1:55151`

3. Nhập biệt danh `mynick`

4. Trong khung nhập, nhập `hi, all ！`

*Dữ liệu gói bắt được cuối cùng như sau:*

```plaintext
/*
 * Bước thiết lập kết nối TCP lần thứ nhất
 * Cổng cục bộ 60653 của trình duyệt gửi gói SYN đến cổng 7272 ở xa
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
...............

/*
 * Bước thiết lập kết nối TCP lần thứ hai
 * Cổng 7272 ở xa gửi gói SYN+ACK đến cổng 60653 của trình duyệt 
 */
17:50:00.523935 IP 127.0.0.1.7272 > 127.0.0.1.60653: Flags [S.], seq 692696454, ack 3524290971, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679666,nop,wscale 7], length 0
E..<..@.@.<..........h..)I....i......0....@....
...............

/*
 * Bước thiết lập kết nối TCP lần thứ ba, hoàn tất kết nối TCP
 * Cổng cục bộ 60653 của trình duyệt gửi gói ACK đến cổng 7272 của xa
 */
17:50:00.523948 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [.], ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 0
E..4.i@.@.HX...........h..i.)I.......(.....
........

/*
 * Thiết lập websocket
 * Cổng cục bộ 60653 của trình duyệt gửi dữ liệu yêu cầu thiết lập websocket đến cổng 7272 của xa
 */
17:50:00.524412 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [P.], seq 1:716, ack 1, win 256, options [nop,nop,TS val 28679666 ecr 28679666], length 715
E....j@.@.E............h..i.)I.............
.........GET / HTTP/1.1
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

// ... và còn nhiều bước khác
```

*Trên đây là tất cả các yêu cầu đăng nhập và phát biểu, có tổng cộng hai máy khách trình duyệt.*

*Dữ liệu các gói có ký tự ```[S]``` đại diện cho yêu cầu ```SYN``` (bắt đầu yêu cầu kết nối); ký tự ```[.]``` đại diện cho Phản hồi ```ACK```, cho biết yêu cầu đã nhận được từ bên đối tác; ký tự ```[P]``` đại diện cho việc gửi dữ liệu; ```[P.]``` đại diện cho [P] + [.]*

*Nếu dữ liệu trên cổng là dữ liệu nhị phân, bạn có thể xem nó dưới dạng thập lục phân:* ```tcpdump -XAns 4096 -iany port 7272```
