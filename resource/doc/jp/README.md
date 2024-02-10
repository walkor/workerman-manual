# 序文

**Workerman、高性能なPHPアプリケーションコンテナ**

## Workermanとは何ですか？

Workermanは、純粋なPHPで開発されたオープンソースの高性能PHPアプリケーションコンテナです。

Workermanは車輪の再発明ではありません。これはMVCフレームワークではなく、より低レベルで一般的なサービスフレームワークであり、これを使用してTCPプロキシ、VPNプロキシ、ゲームサーバ、メールサーバ、FTPサーバ、さらにはPHP版のRedis、PHP版のデータベース、PHP版のNginx、PHP版のPHP-FPMなどを開発できます。WorkermanはPHP領域での革新と言えるでしょう。これにより、開発者はPHPだけでWebを作るという束縛から完全に解放されます。

実際、WorkermanはPHP版のNginxに似ており、コアもマルチプロセス+Epoll+ノンブロッキングIOです。Workermanの各プロセスは数万の同時接続を維持できます。常駐メモリを利用し、Apache、Nginx、PHP-FPMなどのコンテナに依存せず、非常に高いパフォーマンスを持っています。また、TCP、UDP、UNIXソケットをサポートし、長期接続をサポートし、WebSocket、HTTP、WSS、HTTPSなどの通信プロトコルやさまざまなカスタムプロトコルをサポートしています。タイマー、非同期ソケットクライアント、非同期Redis、非同期HTTP、非同期メッセージキューなど多数の高性能コンポーネントを備えています。

## Workermanのいくつかの応用分野

Workermanは従来のMVCフレームワークとは異なり、Web開発にだけ使われるものではなく、即時通信、IoT、ゲーム、サービスガバナンス、他のサーバやミドルウェアなど幅広い応用分野に向けられています。これにより、PHP開発者の視野が大幅に広がります。これらの分野のPHP開発者はまれであり、PHP領域で技術的な優位性を持ちたい場合、毎日のCRUD作業に満足せず、アーキテクトやテックリードに進んだり、Workermanは非常に学びがあるフレームワークです。開発者には単に使用するだけでなく、Workermanをベースに独自のオープンソースプロジェクトを開発することで、スキルを向上させて影響力を高めることをお勧めします。たとえば、[Beanbunマルチプロセスネットワーククローラーフレームワーク](https://github.com/kiddyuchina/Beanbun)は、その立ち上げたばかりで多くの賞賛を受けています。

Workermanのいくつかの応用分野は以下の通りです：

1. 即時通信
   例：ウェブのリアルタイムチャット、リアルタイムメッセージ送信、WeChat Mini Program、モバイルアプリのメッセージ送信、PCソフトウェアのメッセージ送信など
   [[例 workerman-chatチャットルーム](https://www.workerman.net/workerman-chat)、[Webメッセージ送信](https://www.workerman.net/web-sender)、[Workerman-Todpoleチャットルーム](https://www.workerman.net/workerman-todpole)]


2. IoT
   例：Workermanを用いたプリンター通信、マイコン通信、スマートウォッチ、スマートホーム、シェアサイクルなど
   [お客様の事例：易联云、易泊时代等]

3. ゲームサーバ
   例：トランプゲーム、MMORPGゲームなど
   [[例 browserquest-php](https://www.workerman.net/browserquest)]

4. HTTPサービス
   例：高性能なHTTPインターフェース、高性能なウェブサイト。HTTP関連のサービスやサイトを構築する場合は、[webman](https://github.com/walkor/webman)を強くお勧めします。

5. SOAサービス化
   Workermanを使用して、既存の業務機能ユニットをサービスとしてまとめ、システムの疎結合、容易なメンテナンス、高い可用性、拡張性を実現します。
   [[例 workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc)、[workerman-thrift](https://github.com/walkor/workerman-thrift)]

6. その他のサーバーソフトウェア
   例 [GatewayWorker](https://www.workerman.net/doc/gateway-worker)、[PHPSocket.IO](https://www.workerman.net/phpsocket_io)、[HTTPプロキシ](https://github.com/walkor/php-http-proxy)、[sock5プロキシ](https://github.com/walkor/php-socks5)、[分散通信コンポーネント](https://github.com/walkor/Channel)、[分散データ共有コンポーネント](https://github.com/walkor/GlobalData)、[メッセージキュー](https://github.com/walkor/workerman-queue)、DNSサーバ、Webサーバ、CDNサーバ、FTPサーバなど

7. コンポーネント
   例：[非同期Redis](components/workerman-redis.md)、[非同期HTTPクライアント](components/workerman-http-client.md)、[IoT MQTTクライアント](components/workerman-mqtt.md)、[メッセージキュー workerman/redis-queue](components/workerman-redis-queue.md)、[workerman/stomp](components/workerman-stomp.md)、[workerman/rabbitmq](components/workerman-rabbitmq.md) 、[ファイル監視コンポーネント](components/file-monitor.md)など、さまざまなサードパーティー開発のコンポーネントフレームワークがあります

伝統的なMVCフレームワークでは、上記のような機能を実現するのが難しいため、そのためにWorkermanが誕生したのです。

## Workermanの考え方

シンプル、安定、高性能、分散。

### **シンプル**
小さければ美しい。Workermanのコアは非常にシンプルで、わずかなPHPファイルのみを公開し、学習が非常に簡単です。その他の機能は、コンポーネントとして拡張可能です。

Workermanには完全なドキュメント、権威あるウェブサイト、活発なコミュニティ、複数の1000人規模のQQグループ、多数の高性能コンポーネント、そして多くの例があり、これにより開発者は簡単に使用できます。

### **安定**
Workermanは数年前にオープンソース化され、多くの上場企業で大規模に使用されています。非常に安定しており、一部のサービスは2年以上再起動せずに高速で実行されています。コアダンプ、メモリリーク、バグはありません。

### **高性能**
常駐メモリを利用しているため、apache/nginx/php-fpmに依存せず、リクエストごとにすべてを初期化して破棄するコストがなく、伝統的なMVCフレームワークよりも数十倍の性能を持っています。PHP7では、ab負荷テストによるQPSが単独のnginxを上回るほどの高性能です。

### **分散**
今や一人での戦いではなく、一台のサーバの性能がいかに優れていても、分散マルチサーバの展開が最も重要です。Workermanは直接[GatewayWorkerフレームワーク](https://doc2.workerman.net)を提供し、サーバの追加は簡単な設定と起動だけで済み、ビジネスコードの変更は必要ありません。システムの負荷容量が倍増します。TCP長期接続アプリケーションを開発する場合は、[GatewayWorker](https://doc2.workerman.net)を直接使用することをお勧めします。これはWorkermanをラップし、長期接続アプリケーションにより豊富なインターフェースと強力な分散処理能力を提供しています。

## 本マニュアルの対象範囲

Workerman 3.x - 4.x バージョン

## Windowsユーザー（必読）

WorkermanはLinuxシステムとWindowsシステムの両方をサポートしています。Windows版のWorkermanは**任意の拡張機能に依存しません**。PHP環境変数を適切に設定するだけで使用できます。**Windows版のWorkermanのインストールと注意事項については、[Windowsユーザー必読](https://www.workerman.net/windows)を参照してください。**

## クライアント

WorkerManの通信プロトコルはオープンであり、カスタマイズ可能です。したがって、理論上WorkerManは任意のプロトコルを使用する任意のプラットフォームのクライアントと通信できます。ユーザーがクライアントを開発するときは、対応する通信プロトコルに従い、サーバとの通信を行います。
