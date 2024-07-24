# 注意事項
## タイマーの使用に関する注意事項
1. タイマーは```onXXXX```コールバック内でのみ追加することができます。グローバルなタイマーは```onWorkerStart```コールバックで設定することをお勧めし、特定の接続に対するタイマーは```onConnect```で設定することをお勧めします。

2. 追加されたタイマータスクは、現在のプロセスで実行されます（新しいプロセスやスレッドは開始されません）。タスクが重い（特にネットワークI/Oが関与する場合など）場合、このプロセスがブロックされ、他のビジネスを一時的に処理できなくなります。したがって、時間のかかるタスクは別のプロセスで実行することをお勧めします。例えば、1つまたは複数のWorkerプロセスを起動します。

3. 現在のプロセスが他のビジネスに忙しい場合や、タスクが予想された時間に完了していない場合、次の実行サイクルに入るとそのタスクが完了するまで待機し、タイマーが予想される時間間隔で実行されないことがあります。つまり、現在のプロセスのビジネスは直列で実行され、複数のプロセスの場合、プロセス間のタスク実行が並行して行われます。

4. 複数のプロセスにタイマータスクを設定すると、競合問題が発生する可能性があります。例えば、以下のコードは1秒ごとに5回出力されます。
```php
$worker = new Worker();
// 5つのプロセス
$worker->count = 5;
$worker->onWorkerStart = function(Worker $worker) {
    // 5つのプロセスそれぞれにこのようなタイマーがあります
    Timer::add(1, function(){
        echo "hi\r\n";
    });
};
Worker::runAll();
```
1つのプロセスでのみタイマーを実行したい場合は、[Timer::addの例2](add.md)を参照してください。

5. 誤差が約1ミリ秒発生する可能性があります。

6. タイマーはプロセス間で削除することはできません。例えば、プロセスaで設定したタイマーは、プロセスbで直接Timer::delインターフェースを使って削除することはできません。

7. 異なるプロセス間でタイマーIDが重複する可能性がありますが、同じプロセス内で生成されるタイマーIDは重複しません。

8. システム時間を変更すると、タイマーの動作に影響を与える可能性があります。したがって、システム時間を変更したらrestartして再起動することをお勧めします。