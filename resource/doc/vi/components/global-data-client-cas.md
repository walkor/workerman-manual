# CAS

**``` (Yêu cầu phiên bản Workerman >= 3.3.0) ```**
```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
Thay thế nguyên tử, sử dụng ```$new_value``` thay thế ```$old_value```.
Chỉ khi giá trị tương ứng với khóa này không bị thay đổi bởi các khách hàng khác kể từ lần lấy giá trị cuối cùng của khách hàng hiện tại, giá trị mới mới được ghi.

## Tham số

 ``` $key ```

Giá trị khóa. (ví dụ ```$global->abc```, ```abc``` chính là giá trị khóa)

 ``` $old_value ```

Dữ liệu cũ

 ``` $new_value ```

Dữ liệu mới

## Giá trị trả về
Trả về true nếu thay thế thành công, ngược lại trả về false.

## Giải thích:

Khi nhiều tiến trình đồng thời hoạt động trên cùng một biến chia sẻ, đôi khi cần xem xét vấn đề đồng thời.

Ví dụ, A và B hai tiến trình đều thêm một thành viên vào danh sách người dùng.
Tiến trình A và B hiện tại cùng cho ```$global->user_list = array(1,2,3)```.
Tiến trình A thao tác biến ```$global->user_list```, thêm một người dùng 4.
Tiến trình B thao tác biến ```$global->user_list```, tăng thêm một người dùng 5.
Tiến trình A đặt biến ```$global->user_list = array(1,2,3,4)``` thành công.
Tiến trình B đặt biến ```$global->user_list = array(1,2,3,5)``` thành công.
Lúc này biến được thiết lập bởi tiến trình B sẽ ghi đè lên biến được thiết lập bởi tiến trình A, dẫn đến mất dữ liệu.

Trên đây là do thao tác đọc và thiết lập không phải là một thao tác nguyên tử, dẫn đến vấn đề đồng thời.
Để giải quyết vấn đề đồng thời này, có thể sử dụng giao diện thay thế nguyên tử (CAS).
Giao diện CAS sẽ dựa trên ```$old_value``` để xác định xem giá trị này đã được thay đổi bởi tiến trình khác hay chưa,
nếu đã được thay đổi, thì nó sẽ không thay thế và trả về false. Nếu không, thay thế và trả về true.
Xem ví dụ bên dưới.

 **Chú ý:** 
Một số dữ liệu chia sẻ bị ghi đè đồng thời không phải là vấn đề, chẳng hạn như hệ thống đấu giá giá cao nhất hiện tại của một số mục đấu giá, số lượng hàng tồn kho hiện tại của một số sản phẩm.

## Ví dụ

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// Khởi tạo danh sách
$global->user_list = array(1,2,3);

// Thêm một giá trị nguyên tử vào user_list
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
