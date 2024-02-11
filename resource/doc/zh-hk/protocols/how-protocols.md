## 如何自訂協議

實際上制定自己的協議是比較簡單的事情。簡單的協議一般包含兩部分:
 * 區分數據邊界的標識
 * 數據格式定義

## 一個例子

### 協議定義
這裡假設區分數據邊界的標識為換行符"\n"（注意請求數據本身內部不能包含換行符），數據格式為Json，例如下面是一個符合這個規則的請求包。

<pre>
{"type":"message","content":"hello"}

</pre>

注意上面的請求數據末尾有一個換行字符(在PHP中用**雙引號**字符串"\n"表示)，代表一個請求的結束。

### 實現步驟
在 Workerman 中如果要實現上面的協議，假設協議的名字叫 JsonNL，所在項目為 MyApp，則需要以下步驟

1、協議文件放到項目的 Protocols 文件夾，例如文件 MyApp/Protocols/JsonNL.php

2、實現 JsonNL 類，以```namespace Protocols;```為命名空間，必須實現三個靜態方法分別為 input、encode、decode

注意：workerman 會自動調用這三個靜態方法，用來實現分包、解包、打包。具體流程參考下面執行流程說明。

### workerman 與協議類交互流程
1、假設客戶端發送一個數據包給服務端，服務端收到數據(可能是部分數據)後會立刻調用協議的```input```方法，用來檢測這包的長度，```input```方法返回長度值```$length```給 workerman 框架。
2、workerman 框架得到這個```$length```值後判斷當前數據緩衝區中是否已經接收到```$length```長度的數據，如果沒有就會繼續等待數據，直到緩衝區中的數據長度不小於```$length```。
4、緩衝區的數據長度足夠後，workerman 就會從緩衝區截取出```$length```長度的數據(即**分包**)，並調用協議的```decode```方法**解包**，解包後的數據為```$data```。
3、解包後 workerman 將數據```$data```以回調```onMessage($connection, $data)```的形式傳遞給業務，業務在 onMessage 里就可以使用```$data```變量得到客戶端發來的完整並且已經解包的數據了。
4、當```onMessage```里業務需要通過調用```$connection->send($buffer)```方法給客戶端發送數據時，workerman會自動利用協議的```encode```方法將```$buffer```**打包**後再發給客戶端。

### 具體實現

**MyApp/Protocols/JsonNL.php的實現**

```php
namespace Protocols;
class JsonNL
{
    /**
     * 檢查包的完整性
     * 如果能夠得到包長，則返回包的在 buffer 中的長度，否則返回0繼續等待數據
     * 如果協議有問題，則可以返回false，當前客戶端連接會因此斷開
     * @param string $buffer
     * @return int
     */
    public static function input($buffer)
    {
        // 獲得換行字符"\n"位置
        $pos = strpos($buffer, "\n");
        // 沒有換行符，無法得知包長，返回0繼續等待數據
        if($pos === false)
        {
            return 0;
        }
        // 有換行符，返回當前包長（包含換行符）
        return $pos+1;
    }

    /**
     * 打包，當向客戶端發送數據的時候會自動調用
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // json序列化，並加上換行符作為請求結束的標記
        return json_encode($buffer)."\n";
    }

    /**
     * 解包，當接收到的數據字節數等於 input 返回的值（大於0的值）自動調用
     * 並傳遞給 onMessage回調函數的$data參數
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // 去掉換行，還原成數組
        return json_decode(trim($buffer), true);
    }
}
```

至此，JsonNL協議實現完畢，可以在 MyApp 項目中使用，使用方法例如下面

文件：MyApp\start.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data就是客戶端傳來的數據，數據已經經過JsonNL::decode處理過
    echo $data;
    
    // $connection->send的數據會自動調用JsonNL::encode方法打包，然後發往客戶端
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
...
```

> **提示**
> workerman 會嘗試加載`Protocols`命名空間下的協議，例如`new Worker('JsonNL://0.0.0.0:1234')`會嘗試加載`Protocols\JsonNL`協議。
> 如果報錯`Class 'Protocols\JsonNL' not found`，請參考[自動加載](../faq/autoload.md)實現自動加載。

### 協議接口說明
在 Workerman 中開發的協議類必須實現三個靜態方法，input、encode、decode，協議接口說明見Workerman/Protocols/ProtocolInterface.php，定義如下：

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

/**
 * Protocol interface
* @author walkor <walkor@workerman.net>
 */
interface ProtocolInterface
{
    /**
     * 用於在接收到的 recv_buffer 中分包
     *
     * 如果可以在$recv_buffer中得到請求包的長度則返回整個包的長度
     * 否則返回0，表示需要更多的數據才能得到當前請求包的長度
     * 如果返回false或者負數，則代表錯誤的請求，則連接會斷開
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * 用於請求解包
     *
     * input返回值大於0，並且 Workerman 收到了足夠的數據，則自動調用decode
     * 然後觸發onMessage回調，並將decode解碼後的數據傳遞給onMessage回調的第二個參數
     * 也就是說當收到完整的客戶端請求時，會自動調用decode解碼，無需業務代碼中手動調用
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     * 用於請求打包
     *
     * 當需要向客戶端發送數據即調用$connection->send($data);時
     * 會自動把$data用encode打包一次，變成符合協議的數據格式，然後再發送給客戶端
     * 也就是說發送給客戶端的數據會自動encode打包，無需業務代碼中手動調用
     * @param ConnectionInterface $connection
     * @param mixed $data
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## 注意：
Workerman中沒有嚴格要求協議類必須基於 ProtocolInterface實現，實際上協議類只要類包含了input、encode、decode三個靜態方法即可。
