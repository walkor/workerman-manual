# 몇 가지 예시

## 예시 1

### 프로토콜 정의
  * 전체 데이터 패킷 길이를 저장하기 위해 헤더는 10바이트로 고정되며 부족한 부분은 0으로 채워짐
  * 데이터 형식은 xml

### 데이터 패킷 샘플
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
여기서 0000000121은 전체 데이터 패킷의 길이를 나타내며, xml 데이터 형식의 패킷 내용이 바로 뒤따라 옴

### 프로토콜 구현
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // 10바이트 미만이면 0 반환하여 데이터 대기
            return 0;
        }
        // 헤더 데이터 길이+패킷 바디 길이가 포함된 전체 길이 반환
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // 요청 패킷 바디
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        // 패킷 바디와 헤더 길이
        $total_length = strlen($xml_string)+10;
        // 길이 부분을 10자리로 맞추고 부족한 부분은 0으로 채움
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        // 데이터 반환
        return $total_length_str . $xml_string;
    }
}
```

## 예시 2

### 프로토콜 정의
  * 4바이트 네트워크 바이트 순서의 unsigned int를 사용하여 전체 패킷 길이를 표시
  * 데이터 부분은 Json 문자열임

### 데이터 패킷 샘플
<pre>
****{"type":"message","content":"hello all"}
</pre>
여기서 첫 4바이트 *는 보이지 않는 문자로 표시된 네트워크 바이트 순서의 unsigned int 데이터를 나타내며, 이후에는 Json 데이터 형식의 패킷 내용이 옴

### 프로토콜 구현
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        // 받은 데이터가 4바이트 미만이면 패킷 길이를 알 수 없으므로 0 반환하여 데이터 대기
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        // unpack 함수를 이용하여 첫 4바이트를 숫자로 변환하고, 이 값이 전체 데이터 패킷 길이임
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        // 첫 4바이트를 제거하여 패킷 바디의 Json 데이터를 얻음
        $body_json_str = substr($recv_buffer, 4);
        // json 디코딩
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        // Json 인코딩하여 패킷 바디 얻음
        $body_json_str = json_encode($data);
        // 전체 패킷 길이 계산, 헤더 4바이트 + 바디 바이트 수
        $total_length = 4 + strlen($body_json_str);
        // 패킷을 패킹하여 반환
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## 예시 3 (이진 프로토콜을 사용한 파일 업로드)

### 프로토콜 정의
```C
struct
{
  unsigned int total_len;  // 전체 패킷 길이, 빅 엔디안 네트워크 바이트 순서
  char         name_len;   // 파일 이름 길이
  char         name[name_len]; // 파일 이름
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // 파일 데이터
}
```
### 프로토콜 샘플
<pre>*****logo.png******************</pre>
여기서 처음 4바이트 *은 보이지 않는 문자로 표시된 네트워크 바이트 순서의 unsigned int 데이터를 나타내며, 그 뒤에 있는 5번째 *은 파일 이름 길이를 나타내는 바이트이고, 직후에는 파일 이름의 원시 바이너리 데이터가 옴

### 프로토콜 구현
```php
namespace Protocols;
class BinaryTransfer
{
    // 프로토콜 헤더 길이
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        // 프로토콜 헤더 길이보다 작으면 계속해서 데이터 기다림
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        // 언팩
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // 패킷 길이 반환
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        // 언팩
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // 파일 이름 길이
        $name_len = $package_data['name_len'];
        // 데이터 스트림에서 파일 이름을 얻음
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        // 데이터 스트림에서 파일 바이너리 데이터를 얻음
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // 필요에 따라 클라이언트에 전송할 데이터를 인코딩할 수 있음, 여기서는 텍스트를 그대로 반환함
        return $data;
    }
}
```

### 서버 프로토콜 사용 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// 파일을 tmp에 저장
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### 클라이언트 파일 client.php (여기서는 PHP로 클라이언트 업로드를 시뮬레이션함)
```php
<?php
/** 파일 업로드 클라이언트 **/
// 업로드 주소
$address = "127.0.0.1:8333";
// 업로드 파일 경로 매개변수 확인
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// 업로드 파일 경로
$file_to_transfer = trim($argv[1]);
// 업로드할 파일이 로컬에 없음
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// 소켓 연결 생성
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// 블로킹 모드로 설정
stream_set_blocking($client, 1);
// 파일 이름
$file_name = basename($file_to_transfer);
// 파일 이름 길이
$name_len = strlen($file_name);
// 파일 바이너리 데이터
$file_data = file_get_contents($file_to_transfer);
// 프로토콜 헤더 길이 4바이트 패킷 길이 + 1바이트 파일 이름 길이
$PACKAGE_HEAD_LEN = 5;
// 프로토콜 패킷
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// 업로드 실행
fwrite($client, $package);
// 결과 출력
echo fread($client, 8192),"\n";
```

### 클라이언트 사용 예시
명령 프롬프트에서 ```php client.php <파일 경로>```를 실행

예: ```php client.php abc.jpg```
## 예제 4 (텍스트 프로토콜을 사용한 파일 업로드)

### 프로토콜 정의

json+줄 바꿈, json에 파일 이름과 base64_encode로 인코드된 파일 데이터가 포함되어 있습니다 (크기는 1/3 증가합니다).

### 프로토콜 샘플

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

끝에 줄 바꿈 문자가 있는 것에 유의하시기 바랍니다. PHP에서는 이를 이중 인용부호 문자 ```"\n"```으로 표시합니다.

### 프로토콜 구현
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
        // 언패킹
        $package_data = json_decode(trim($recv_buffer), true);
        // 파일 이름 가져오기
        $file_name = $package_data['file_name'];
        // base64_encode로 인코드된 파일 데이터 가져오기
        $file_data = $package_data['file_data'];
        // base64_decode를 사용하여 원래의 이진 파일 데이터로 복원
        $file_data = base64_decode($file_data);
        // 데이터 반환
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // 클라이언트에 전송할 데이터를 필요에 따라 인코드할 수 있습니다. 여기서는 텍스트를 그대로 반환하는 것입니다.
        return $data;
    }
}

```

### 서버 측 프로토콜 사용 예제
설명: 바이너리 업로드 방식과 동일한 방식으로 작성되어 있으며, 거의 어떤 비즈니스 코드 수정 없이 프로토콜을 전환할 수 있습니다.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// tmp에 파일 저장
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### 클라이언트 파일 textclient.php (여기서는 PHP를 사용하여 클라이언트 업로드 시뮬레이션)
```php
<?php
/** 파일 업로드 클라이언트 **/
// 업로드 주소
$address = "127.0.0.1:8333";
// 업로드 파일 경로 매개변수 확인
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// 파일을 업로드할 경로
$file_to_transfer = trim($argv[1]);
// 업로드하는 파일이 로컬에 없는 경우
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// 소켓 연결 설정
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// 파일 이름
$file_name = basename($file_to_transfer);
// 파일 이진 데이터
$file_data = file_get_contents($file_to_transfer);
// base64 인코딩
$file_data = base64_encode($file_data);
// 데이터 패키지
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// 프로토콜 패키지 json+줄 바꿈
$package = json_encode($package_data)."\n";
// 업로드 실행
fwrite($client, $package);
// 결과 출력
echo fread($client, 8192),"\n";
```

### 클라이언트 사용 예제
명령 프롬프트에서 ```php textclient.php <파일 경로>```를 실행합니다.

예: ```php textclient.php abc.jpg```
