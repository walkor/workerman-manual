# 一些範例

## 範例一

### 協議定義
  * 首部固定10個字節長度用來保存整個數據包長度，位數不夠補0
  * 數據格式為xml

### 數據包樣本
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
其中0000000121代表整個數據包長度，後面緊跟xml數據格式的包體內容

### 協議實現
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // 不夠10字節，返回0繼續等待數據
            return 0;
        }
        // 返回包長，包長包含 頭部數據長度+包體長度
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // 請求包體
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        // 包體+包頭的長度
        $total_length = strlen($xml_string)+10;
        // 長度部分湊足10字節，位數不夠補0
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        // 返回數據
        return $total_length_str . $xml_string;
    }
}
```

## 範例二

### 協議定義
  * 首部4字節網絡字節序unsigned int，標記整個包的長度
  * 數據部分為Json字符串

### 數據包樣本
<pre>
****{"type":"message","content":"hello all"}
</pre>

其中首部四字節*號代表一個網絡字節序的unsigned int數據，為不可見字符，緊接著是Json的數據格式的包體數據

### 協議實現
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        // 接收到的數據還不夠4字節，無法得知包的長度，返回0繼續等待數據
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        // 利用unpack函數將首部4字節轉換成數字，首部4字節即為整個數據包長度
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        // 去掉首部4字節，得到包體Json數據
        $body_json_str = substr($recv_buffer, 4);
        // json解碼
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        // Json編碼得到包體
        $body_json_str = json_encode($data);
        // 計算整個包的長度，首部4字節+包體字節數
        $total_length = 4 + strlen($body_json_str);
        // 返回打包的數據
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## 範例三（使用二進制協議上傳檔案）
### 協議定義
```C
struct
{
  unsigned int total_len;  // 整個包的長度，大端網絡字節序
  char         name_len;   // 檔案名的長度
  char         name[name_len]; // 檔案名
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // 檔案數據
}
```
### 協議樣本

<pre> *****logo.png****************** </pre>

其中首部四字節\*號代表一個網絡字節序的unsigned int數據，為不可見字符，第5個\*是用一個字節存儲檔案名長度，緊接著是檔案名，接著是原始的二進制檔案數據

### 協議實現
```php
namespace Protocols;
class BinaryTransfer
{
    // 協議頭長度
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        // 如果不夠一個協議頭的長度，則繼續等待
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        // 解包
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // 返回包長
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        // 解包
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // 檔案名長度
        $name_len = $package_data['name_len'];
        // 從數據流中截取出檔案名
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        // 從數據流中截取出檔案二進制數據
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // 可以根據自己的需要編碼發送給客戶端的數據，這裡只是當做文本原樣返回
        return $data;
    }
}
```

### 服務端協議使用示例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// 保存檔案到tmp下
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### 客戶端檔案 client.php （這裡用php模擬客戶端上傳）
```php
<?php
/** 上傳檔案客戶端 **/
// 上傳地址
$address = "127.0.0.1:8333";
// 檢查上傳檔案路徑參數
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// 上傳檔案路徑
$file_to_transfer = trim($argv[1]);
// 上傳的檔案本地不存在
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// 建立socket連接
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// 設置成阻塞
stream_set_blocking($client, 1);
// 檔案名
$file_name = basename($file_to_transfer);
// 檔案名長度
$name_len = strlen($file_name);
// 檔案二進制數據
$file_data = file_get_contents($file_to_transfer);
// 協議頭長度 4字節包長+1字節檔案名長度
$PACKAGE_HEAD_LEN = 5;
// 協議包
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// 執行上傳
fwrite($client, $package);
// 打印結果
echo fread($client, 8192),"\n";
```  

### 客戶端使用示例
命令行中運行 ```php client.php <檔案路徑>```  

例如 ```php client.php abc.jpg```
## 範例四（使用文本協議上傳檔案）

### 協議定義

json+換行，json中包含了檔案名稱以及base64_encode編碼（會增大1/3的體積）的檔案數據

### 協議範本

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

注意末尾為一個換行符，在PHP中用雙引號字元```"\n"```標識

### 協議實現
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
        // 解包
        $package_data = json_decode(trim($recv_buffer), true);
        // 取出檔案名稱
        $file_name = $package_data['file_name'];
        // 取出base64_encode後的檔案數據
        $file_data = $package_data['file_data'];
        // base64_decode還原回原來的二進位檔案數據
        $file_data = base64_decode($file_data);
        // 返回數據
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // 可以根據自己的需要編碼發送給客戶端的數據，這裡只是當做文本原樣返回
        return $data;
    }
}

```


### 服務端協議使用範例
說明：寫法與二進制上傳寫法一樣，即能做到幾乎不用改動任何業務代碼便可以切換協議

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// 保存檔案到tmp下
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```


### 客戶端檔案 textclient.php （這裡用php模擬客戶端上傳）
```php
<?php
/** 上傳檔案客戶端 **/
// 上傳地址
$address = "127.0.0.1:8333";
// 檢查上傳檔案路徑參數
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// 上傳檔案路徑
$file_to_transfer = trim($argv[1]);
// 上傳的檔案本地不存在
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// 建立socket連接
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// 檔案名
$file_name = basename($file_to_transfer);
// 檔案二進位數據
$file_data = file_get_contents($file_to_transfer);
// base64編碼
$file_data = base64_encode($file_data);
// 數據包
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// 協議包 json+換行
$package = json_encode($package_data)."\n";
// 執行上傳
fwrite($client, $package);
// 打印結果
echo fread($client, 8192),"\n";
```


### 客戶端使用範例
命令行中運行 ```php textclient.php <檔案路徑>```

例如 ```php textclient.php abc.jpg```
