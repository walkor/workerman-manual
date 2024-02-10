# Một số ví dụ

## Ví dụ một

### Định nghĩa giao thức
  * Đầu ghi chú có độ dài cố định là 10 byte để lưu trữ độ dài toàn bộ gói dữ liệu, nếu số lượng không đủ thì điền 0
  * Định dạng dữ liệu là xml

### Mẫu gói dữ liệu
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
Trong đó, 0000000121 biểu thị độ dài toàn bộ gói dữ liệu, sau đó là nội dung gói dữ liệu dạng xml

### Triển khai giao thức
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // Không đủ 10 byte, trả về 0 để chờ dữ liệu tiếp theo
            return 0;
        }
        // Trả về độ dài gói, bao gồm độ dài tiêu đề + độ dài gói
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // Gói dữ liệu yêu cầu
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        // Độ dài gói + độ dài tiêu đề
        $total_length = strlen($xml_string)+10;
        // Đảm bảo độ dài cố định 10 byte, nếu số chữ số không đủ sẽ điền 0
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        // Trả về dữ liệu
        return $total_length_str . $xml_string;
    }
}
```

## Ví dụ hai

### Định nghĩa giao thức
  * 4 byte đầu tiên được chuyển đổi sang dạng unsigned int theo thứ tự byte mạng, đánh dấu độ dài của toàn bộ gói
  * Phần dữ liệu là chuỗi Json

### Mẫu gói dữ liệu
<pre>
****{"type":"message","content":"hello all"}
</pre>

Trong đó, bốn byte đầu tiên dấu * biểu thị một dữ liệu unsigned int theo thứ tự byte mạng, là các ký tự không thể nhìn thấy, tiếp theo là dạng dữ liệu chuỗi Json của gói dữ liệu

### Triển khai giao thức
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        // Dữ liệu nhận được không đủ 4 byte, không thể biết được độ dài của gói, trả về 0 để chờ dữ liệu tiếp theo
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        // Sử dụng hàm unpack để chuyển đổi 4 byte đầu tiên sang dạng số, 4 byte đầu tiên chính là độ dài toàn bộ gói dữ liệu
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        // Bỏ 4 byte đầu, lấy dữ liệu Json của gói dữ liệu
        $body_json_str = substr($recv_buffer, 4);
        // Giải mã Json
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        // Mã hóa Json để lấy gói dữ liệu
        $body_json_str = json_encode($data);
        // Tính độ dài toàn bộ gói, 4 byte đầu + số byte trong dữ liệu gói
        $total_length = 4 + strlen($body_json_str);
        // Trả về dữ liệu đã đóng gói
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## Ví dụ ba (sử dụng giao thức nhị phân để tải lên tập tin)

### Định nghĩa giao thức
```C
struct
{
  unsigned int total_len;  // độ dài toàn bộ gói, định dạng byte mạng kích thước 
  char         name_len;   // độ dài tên tập tin
  char         name[name_len]; // tên tập tin
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // dữ liệu tập tin
}
```
### Mẫu gói dữ liệu
<pre> *****logo.png****************** </pre>

Trong đó, bốn byte đầu tiên \* biểu thị một dữ liệu unsigned int theo thứ tự byte mạng, là các ký tự không thể nhìn thấy, ký tự thứ 5 là lưu độ dài tên tập tin bằng một byte, tiếp đó là tên tập tin, tiếp theo là dữ liệu tập tin nhị phân gốc

### Triển khai giao thức
```php
namespace Protocols;
class BinaryTransfer
{
    // Độ dài tiêu đề giao thức
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        // Nếu không đủ độ dài tiêu đề giao thức, tiếp tục chờ đợi
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        // Giải mã gói dữ liệu
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // Trả về độ dài gói
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        // Giải mã gói dữ liệu
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // Độ dài tên tập tin
        $name_len = $package_data['name_len'];
        // Cắt ra tên tập tin từ dữ liệu
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        // Cắt ra dữ liệu nhị phân từ dữ liệu
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // Có thể mã hóa dữ liệu gửi cho máy khách theo yêu cầu của mình, ở đây chỉ trả về dữ liệu văn bản nguyên bản
        return $data;
    }
}
```

### Ví dụ sử dụng giao thức ở phía máy chủ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// Lưu tập tin vào thư mục tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### Máy khách tập tin client.php (ở đây sử dụng php mô phỏng máy khách tải lên)
```php
<?php
/** Máy khách tải lên tập tin **/
// Địa chỉ tải lên
$address = "127.0.0.1:8333";
// Kiểm tra tham số đường dẫn tập tin tải lên
if(!isset($argv[1]))
{
   exit("Sử dụng php client.php \$file_path\n");
}
// Đường dẫn tập tin tải lên
$file_to_transfer = trim($argv[1]);
// Tập tin tải lên không tồn tại
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer không tồn tại\n");
}
// Thiết lập kết nối socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// Đặt chế độ chặn
stream_set_blocking($client, 1);
// Tên tệp
$file_name = basename($file_to_transfer);
// Độ dài tên tệp
$name_len = strlen($file_name);
// Dữ liệu nhị phân tệp
$file_data = file_get_contents($file_to_transfer);
// Độ dài tiêu đề giao thức 4 byte dữ liệu gói + 1 byte tên tệp
$PACKAGE_HEAD_LEN = 5;
// Gói giao thức
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// Tiến hành tải lên
fwrite($client, $package);
// In kết quả
echo fread($client, 8192),"\n";
```

### Ví dụ sử dụng máy khách
Chạy trong dòng lệnh ```php client.php <đường dẫn tệp>```

Ví dụ: ```php client.php abc.jpg```
## Ví dụ 4 (Sử dụng giao thức văn bản để tải lên tệp)

### Định nghĩa giao thức

json + dòng mới, trong json chứa tên tệp và dữ liệu tệp đã mã hóa base64 (sẽ làm tăng kích thước lên 1/3)

### Mẫu giao thức

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

Chú ý: Cuối cùng là một ký tự dòng mới, trong PHP được biểu thị bằng ký tự nháy kép ```"\n"```

### Triển khai giao thức

```php
namespace Protocols;
class TextTransfer
{
    public static function input($recv_buffer)
    {
        $recv_len = strlen($recv_buffer);
        if($recv_buffer[$recv_len-1] !== "\n")
        {
            return 0;
        }
        return strlen($recv_buffer);
    }

    public static function decode($recv_buffer)
    {
        // Giải mã
        $package_data = json_decode(trim($recv_buffer), true);
        // Lấy tên tệp
        $file_name = $package_data['file_name'];
        // Lấy dữ liệu tệp đã mã hóa base64
        $file_data = $package_data['file_data'];
        // Giải mã base64 để khôi phục dữ liệu tệp nhị phân ban đầu
        $file_data = base64_decode($file_data);
        // Trả về dữ liệu
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // Có thể mã hóa dữ liệu gửi cho máy khách theo nhu cầu của bạn, ở đây chỉ trả về nguyên văn bản
        return $data;
    }
}
```

### Ví dụ sử dụng giao thức trên máy chủ

Giải thích: Cách viết giống như cách tải tệp nhị phân, nghĩa là có thể chuyển đổi giao thức mà không cần thay đổi mã nguồn kinh doanh nào

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// Lưu tệp vào thư mục tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("Tải lên thành công. Đường dẫn lưu trữ $save_path");
};

Worker::runAll();
```

### Ví dụ sử dụng máy khách tệp textclient.php (sử dụng PHP mô phỏng máy khách tải lên)

```php
<?php
/** Máy khách tải lên tệp **/
// Địa chỉ tải lên
$address = "127.0.0.1:8333";
// Kiểm tra tham số đường dẫn tệp tải lên
if(!isset($argv[1]))
{
   exit("Sử dụng php client.php \$file_path\n");
}
// Đường dẫn tệp tải lên
$file_to_transfer = trim($argv[1]);
// Tệp tải lên không tồn tại trên máy
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer không tồn tại\n");
}
// Thiết lập kết nối socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// Tên tệp
$file_name = basename($file_to_transfer);
// Dữ liệu nhị phân của tệp
$file_data = file_get_contents($file_to_transfer);
// Mã hóa base64
$file_data = base64_encode($file_data);
// Gói dữ liệu
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// Gói giao thức json + Enter
$package = json_encode($package_data)."\n";
// Thực hiện tải lên
fwrite($client, $package);
// In kết quả
echo fread($client, 8192),"\n";
```

### Ví dụ sử dụng máy khách

Chạy trong dòng lệnh ```php textclient.php <đường dẫn tệp>```

Ví dụ: ```php textclient.php abc.jpg```
