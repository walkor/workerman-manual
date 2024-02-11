# 実行状態の確認

```php start.php status```を実行すると、Workermanの実行状態を確認できます。以下のような結果が表示されます。

```plaintext
----------------------------------------------GLOBAL STATUS----------------------------------------------------
Workerman version:3.5.13          PHP version:5.5.9-1ubuntu4.24
start time:2018-02-03 11:48:20   run 112 days 2 hours   
load average: 0, 0, 0            event-loop:\Workerman\Events\Event
4 workers       11 processes
worker_name        exit_status      exit_count
ChatBusinessWorker 0                0
ChatGateway        0                0
Register           0                0
WebServer          0                0
----------------------------------------------PROCESS STATUS---------------------------------------------------
pid	memory  listening                worker_name        connections send_fail timers  total_request qps    status
18306	2.25M   none                     ChatBusinessWorker 5           0         0       11            0      [idle]
18307	2.25M   none                     ChatBusinessWorker 5           0         0       8             0      [idle]
18308	2.25M   none                     ChatBusinessWorker 5           0         0       3             0      [idle]
18309	2.25M   none                     ChatBusinessWorker 5           0         0       14            0      [idle]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [idle]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [idle]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [idle]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [idle]
18314	1.75M   text://0.0.0.0:1236      Register           8           0         0       8             0      [idle]
18315	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [idle]
18316	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [idle]
----------------------------------------------PROCESS STATUS---------------------------------------------------
Summary	18M     -                        -                  54          0         4       138           0      [Summary]
```

## 説明

### GLOBAL STATUS

この欄から以下の情報が確認できます。

Workermanのバージョン```version:3.5.13```

起動時間 ```2018-02-03 11:48:20```から```run 112 days 2 hours ```

サーバの負荷 ```load average: 0, 0, 0```は、直近1分、5分、15分の平均負荷を示します。

使用しているIOイベントライブラリ```event-loop:\Workerman\Events\Event```

 ```4 workers```（ChatGateway、ChatBusinessWorker、Register、WebServerの4つの種類のワーカープロセス）

 ``` 11 processes ```（合計11個のプロセス）

 ``` worker_name ```（ワーカープロセスの名前）

 ``` exit_status ```（ワーカープロセスの終了ステータス）

 ``` exit_count ```（終了ステータスごとの終了回数）


通常、exit_statusが0の場合は正常な終了を示します。それ以外の値の場合は、プロセスが異常終了し、「WORKER EXIT UNEXPECTED」というエラーメッセージが発生します。エラーメッセージは[Worker::logFile](worker/log-file.md)で指定されたファイルに記録されます。

**一般的なexit_statusおよびその意味は以下の通りです：**

* 0：正常な終了を示します。reload時に発生する値0は正常です。プログラム内でexitまたはdieを呼び出すと、exitコードは0になり、「WORKER EXIT UNEXPECTED」というエラーメッセージが発生します。Workermanでは、ビジネスコードでexitまたはdieステートメントを許可していません。
* 9：プロセスがSIGKILLシグナルで終了しました。これはstopやreload平滑再起動時に発生し、原因は子プロセスが規定の時間内に親プロセスのreloadシグナルに応答しなかった場合に発生します（例：mysql、curlなどの長時間ブロックまたはビジネスの無限ループ）。注意：linuxコマンドラインでkillコマンドを使用して子プロセスにSIGKILLシグナルを送ると、この終了コードが発生します。
* 11：phpがコアダンプを発生させたことを示します。これは不安定な拡張機能が使用されているために発生することが一般的です。対応する拡張機能をphp.iniでコメントアウトにします。また、稀にphpのバグの場合もあります。この場合はphpをアップグレードする必要があります。
* 65280：ビジネスコードに致命的なエラーが発生したことを示します。たとえば、存在しない関数を呼び出した場合、構文エラーなどが該当します。具体的なエラーメッセージは[Worker::logFile](worker/log-file.md)で指定されたファイルに記録され、指定されている場合は[php.ini](https://php.net/manual/zh/ini.list.php)の[error_log](https://php.net/manual/zh/errorfunc.configuration.php#ini.error-log)で見ることができます。
* 64000：ビジネスコードが例外をスローし、その例外がキャッチされなかったことを示します。このことが原因でプロセスが終了します。Workermanがデバッグモードで実行されている場合は例外呼び出しスタックが端末に出力され、デーモンモードで実行されている場合は例外呼び出しスタックが[Worker::stdoutFile](worker/stdout-file.md)で指定されたファイルに記録されます。

## PROCESS STATUS

pid：プロセスID

memory：現在のプロセスが占有するメモリ（phpの実行可能ファイルのメモリは含まれません）

listening：転送層プロトコルおよびリスニングIPとポート。ポートをリスンしない場合はnoneと表示されます。[Workerクラスのコンストラクタ](worker/construct.md)を参照してください。

worker_name：プロセスが実行するサービスの名前。[Workerクラスのname属性](worker/name.md)を参照してください。

connections：プロセスが**現在**持っているTCP接続の数。接続インスタンスにはTcpConnectionやAsyncTcpConnectionのインスタンスが含まれます。この値はリアルタイムの数値であり、累計値ではありません。注意：接続インスタンスがcloseされた場合、対応する数が適切に減少しない場合は、ビジネスコードが$connectionオブジェクトを保持しているため、接続インスタンスが破棄されない可能性があります。

total_request：プロセスが起動してから受け取ったリクエストの総数。ここでいうリクエスト数には、クライアントからのリクエストだけでなく、Workerman内部の通信リクエスト（例：GatewayWorkerアーキテクチャでのGatewayとBusinessWorker間の通信リクエスト）も含まれます。この値は累計値です。

send_fail：プロセスでクライアントにデータを送信できなかった回数。通常、この値が0でない場合は正常な状態であり、[statusのsend_failに関する原因](../faq/about-send-fail.md)を参照してください。この値は累計値です。

timers：プロセスのアクティブなタイマーの数（削除されたタイマーや実行済みの一回限りのタイマーは含まれません）。注意：この機能を使用するには、workermanのバージョンが3.4.7以上である必要があります。この値はリアルタイムの数値であり、累計値ではありません。

qps：プロセスごとの現在のネットワークリクエストの平均数。注意：このオプションが統計されるにはstatus時に```-d```を追加する必要があります。それ以外の場合は0が表示されます。この機能を使用するには、workermanのバージョンが3.5.2以上である必要があります。この値はリアルタイムの数値であり、累計値ではありません。

status：プロセスの状態。idleの場合はアイドル状態を、busyの場合はビジー状態を示します。注意：プロセスが一時的にビジー状態になった場合は正常な状態ですが、プロセスが常にビジー状態になっている場合はビジネスがブロックされたか、またはビジネスの無限ループが発生している可能性があります。[ビジーなプロセスのデバッグ](busy-process.md)のセクションに従って確認してください。この機能を使用するには、workermanのバージョンが3.5.0以上である必要があります。

## 原理
statusスクリプトを実行すると、メインプロセスはすべてのワーカープロセスに```SIGUSR2```シグナルを送信し、その後ステータススクリプトは短いスリープ状態に入り、各ワーカープロセスの状態統計結果を待ちます。このときアイドル状態のワーカープロセスは```SIGUSR2```シグナルを受け取ると直ちに独自の実行状態（接続数、リクエスト数など）を特定のディスクファイルに書き込み、ビジネスロジックを処理中のワーカープロセスはビジネスロジック処理が完了してから自身の状態情報を書き込みます。短いスリープ後、statusスクリプトはディスク中の状態ファイルを読み取り、その結果をコンソールに表示します。
## 注意事项
在查看状态时，您可能会注意到一些进程显示为busy。这是由于该进程正忙于处理业务（例如，业务逻辑可能长时间阻塞在curl或数据库请求上，或者运行大型循环），无法及时上报状态而导致显示为busy。

出现这种情况时，需要检查业务代码，查看是哪里导致了长时间阻塞，并评估阻塞耗时是否在预期范围内。如果不符合预期，则需要依据[调试忙碌进程](busy-process.md)一节来排查业务代码。
