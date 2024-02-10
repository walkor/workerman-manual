# Thành phần chia sẻ biến toàn cầu GlobalData
**``` Ứng dụng yêu cầu phiên bản Workerman >= 3.3.0 ```**

Đường dẫn mã nguồn: https://github.com/walkor/GlobalData

## Lưu ý
GlobalData yêu cầu phiên bản Workerman >= 3.3.0

## Cài đặt

`composer require workerman/globaldata`

## Nguyên lý

Sử dụng phương thức ma thuật ```__set __get __isset __unset``` của PHP để giao tiếp với phía server của GlobalData, thực tế biến được lưu trữ ở phía server của GlobalData. Ví dụ khi một thuộc tính không tồn tại được thiết lập cho một đối tượng client, phương thức ma thuật ```__set``` sẽ được kích hoạt và đối tượng client sẽ gửi yêu cầu đến phía server của GlobalData để lưu trữ một biến. Khi truy cập một biến không tồn tại của đối tượng client, phương thức ```__get``` của lớp sẽ được kích hoạt và đối tượng client sẽ gửi yêu cầu đến phía server của GlobalData để đọc giá trị này, từ đó hoàn tất việc chia sẻ biến giữa các tiến trình.

```php
require_once __DIR__ . '/vendor/autoload.php';

// Kết nối với server Global Data
$global = new GlobalData\Client('127.0.0.1:2207');

// Kích hoạt $global->__isset('somedata') để kiểm tra xem server có lưu trữ giá trị nào với key là somedata hay không
isset($global->somedata);

// Kích hoạt $global->__set('somedata',array(1,2,3)) để thông báo cho server lưu trữ giá trị tương ứng với somedata là mảng(1,2,3)
$global->somedata = array(1,2,3);

// Kích hoạt $global->__get('somedata') để truy vấn giá trị tương ứng với somedata từ server
var_export($global->somedata);

// Kích hoạt $global->__unset('somedata'), thông báo server xóa bỏ somedata và giá trị tương ứng
unset($global->somedata);
```
