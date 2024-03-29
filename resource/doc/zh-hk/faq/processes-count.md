# 應該開多少個進程

## 如何設置進程數
進程數是由```count```屬性決定的（Windows系統不支持進程數設置），例如下面的代碼
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// ## 啟動4個進程對外提供服務 ##
$http_worker->count = 4;

...

```

## 進程數設置需要考慮以下條件
1、CPU核數

2、內存大小

3、業務偏向IO密集還是CPU密集型

## 進程數設置原則

1、每個進程佔用內存之和需要小於總內存（一般來說每個業務進程佔用內存大概40M左右）

2、如果是IO密集型，也就是業務中涉及到一些**阻塞式**IO，比如一般的訪問Mysql、Redis等存儲都是阻塞式訪問的，進程數可以開大一些，如配置成CPU核數的3倍。如果業務中涉及的阻塞等待非常多，可以再適當加大進程數，例如CPU核數的8倍甚至更高。注意**非阻塞式**IO屬於CPU密集型，而不屬於IO密集型。

3、如果是CPU密集型，也就是業務中沒有**阻塞式**IO開銷，例如使用異步IO讀取網絡資源，進程不會被業務代碼阻塞的情況下，可以把進程數設置成和CPU核數一樣

## 進程數設置參考值
如果業務代碼偏向IO密集型，則根據IO密集程度設置進程數，例如CPU核數的3-8倍。

如果業務代碼偏向CPU密集型，則可以將進程數設置成CPU核數。

## 注意
Workerman自身的IO都是非阻塞的，例如```Connection->send```等都是非阻塞的，屬於CPU密集型操作。如果不清楚自己業務偏向於哪種類型，可設置進程數為CPU核數的3倍左右即可。
另外進程數並非越多越好，進程開得太多，進程切換開銷會增大，對性能有一定影響。
