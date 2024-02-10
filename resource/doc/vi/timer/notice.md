# Lưu ý
## Lưu ý khi sử dụng bộ hẹn giờ
1. Chỉ có thể thêm bộ hẹn giờ trong các gọi lại ```onXXXX```. Bộ hẹn giờ toàn cầu được khuyến nghị để thiết lập trong gọi lại ```onWorkerStart```, còn bộ hẹn giờ dành cho một kết nối cụ thể được khuyến nghị để thiết lập trong ```onConnect```.

2. Công việc đặt ra sử dụng bộ hẹn giờ được thực hiện trong quá trình hiện tại (không khởi động quá trình hoặc luồng mới), nếu công việc nặng (đặc biệt là công việc liên quan đến IO mạng) có thể gây ra quá trình này bị chặn và không thể xử lý các công việc khác tạm thời. Do đó, việc thực hiện công việc tốn thời gian trong một quá trình riêng biệt được khuyến nghị, ví dụ như thiết lập một hoặc nhiều quá trình Worker chạy.

3. Khi quá trình hiện tại bận rộn hoặc khi một công việc không hoàn thành trong thời gian dự kiến, sau đó đến chu kỳ chạy tiếp theo, nó sẽ chờ cho công việc hiện tại hoàn thành trước khi chạy, điều này sẽ dẫn đến việc bộ hẹn giờ không chạy theo khoảng thời gian dự kiến. Điều đó có nghĩa là các công việc của quá trình hiện tại được thực hiện theo chuỗi, còn nếu có nhiều quá trình thì các công việc giữa các quá trình sẽ chạy song song.

4. Cần chú ý rằng việc thiết lập nhiều quá trình với bộ hẹn giờ có thể gây ra vấn đề về đồng thời, ví dụ như đoạn mã dưới đây sẽ in ra 5 lần mỗi giây.
```php
$worker = new Worker();
// 5 quá trình
$worker->count = 5;
$worker->onWorkerStart = function(Worker $worker) {
    // 5 quá trình, mỗi quá trình có một bộ hẹn giờ như vậy
    Timer::add(1, function(){
        echo "hi\r\n";
    });
};
Worker::runAll();
```
Nếu chỉ muốn một quá trình chạy bộ hẹn giờ, xem [Example 2](add.md) về Timer::add.

5. Có thể có sai số khoảng 1 mili giây.

6. Bộ hẹn giờ không thể xóa qua các quá trình, ví dụ như bộ hẹn giờ được thiết lập trong quá trình a sẽ không thể xóa trực tiếp bằng giao diện Timer::del trong quá trình b.

7. ID của bộ hẹn giờ có thể trùng nhau giữa các quá trình khác nhau, nhưng ID của bộ hẹn giờ được tạo ra trong cùng một quá trình sẽ không trùng nhau.

8. Thay đổi thời gian hệ thống sẽ ảnh hưởng đến hành vi của bộ hẹn giờ, vì vậy sau khi thay đổi thời gian hệ thống, khuyến nghị là restart để khởi động lại.

