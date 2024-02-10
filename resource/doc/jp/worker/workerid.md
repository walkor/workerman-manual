# id
要求```（workerman >= 3.2.1）```

## 説明:
```php
int Worker::$id
```

現在のworkerプロセスのID番号であり、範囲は```0```から```$worker->count-1```までです。

この属性は、workerプロセスを区別するのに非常に役立ちます。たとえば、1つのworkerインスタンスに複数のプロセスがある場合、開発者はその中の1つのプロセスにのみタイマーを設定したいとしますと、プロセス番号idを認識してそれを実現できます。例えば、workerインスタンスのid番号が0のプロセスにのみタイマーを設定します（例を参照）。

## 注意：

プロセスが再起動された後、id番号の値は変わりません。

プロセスIDの割り当ては各workerインスタンスに基づいています。各workerインスタンスは自身のプロセスに0から番号を割り当てるため、workerインスタンス間のプロセス番号は重複することがありますが、workerインスタンス内のプロセス番号は重複しません。以下は例です：

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// workerインスタンス1に4つのプロセスがあり、プロセスID番号はそれぞれ0、1、2、3になります
$worker1 = new Worker('tcp://0.0.0.0:8585');
// 4つのプロセスを起動する
$worker1->count = 4;
// 各プロセスの起動後、現在のプロセスID番号である $worker1->id を表示
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// workerインスタンス2に2つのプロセスがあり、プロセスID番号はそれぞれ0、1になります
$worker2 = new Worker('tcp://0.0.0.0:8686');
// 2つのプロセスを起動する
$worker2->count = 2;
// 各プロセスの起動後、現在のプロセスID番号である $worker2->id を表示
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// workerを実行
Worker::runAll();
```
出力例
```
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

注意：Windowsシステムはプロセス数countの設定をサポートしていないため、idは0のみです。Windowsシステムでは同じファイルで2つのWorkerリスンを初期化することはサポートされていないため、この例はWindowsシステムでは実行できません。

## 例
1つのworkerインスタンスに4つのプロセスがあり、id番号が0のプロセスにのみタイマーを設定します。

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // id番号が0のプロセスにのみタイマーを設定し、他の1、2、3番のプロセスにはタイマーを設定しません
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4つのworkerプロセス、id番号0のみにタイマーを設定\n";
        });
    }
};
// workerを実行
Worker::runAll();
```
