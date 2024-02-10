# WorkerManを使用してクライアントにデータをプッシュする他のプロジェクトでの利用

**質問:**

普通のWebプロジェクトがあり、このプロジェクトでWorkerManのAPIを呼び出して、クライアントにデータをプッシュしたいと思っています。

**回答:**

**WorkerManベースのプロジェクトの例は以下のリンクを参照してください**

- [Channelコンポーネントのプッシュ例](../components/channel-examples.md)（マルチプロセス/サーバクラスタをサポートし、Channelコンポーネントのダウンロードが必要です）

- [Workerに基づくプッシュ](https://www.workerman.net/q/508)（シングルプロセス、最も簡単な方法）

**webmanに基づく場合は、以下のリンクを参照してください**

- [webman pushプラグイン](https://www.workerman.net/plugin/2)

**GatewayWorkerに基づく場合は、以下のリンクを参照してください**

- [他のプロジェクトからGatewayWorkerを介してプッシュする](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html)（マルチプロセス/サーバクラスタをサポートし、グループ、マルチキャスト、単一の送信をサポート）

**PHPSocket.IOに基づく場合は、以下のリンクを参照してください**

- [ウェブメッセージのプッシュ](https://www.workerman.net/web-sender)（デフォルトはシングルプロセス、socket.ioベース、ブラウザの互換性が最も良い）
