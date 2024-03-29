# 特定のプロセスにリクエストが集中する問題

### 現象
時々、`php start.php status`コマンドで確認すると、リクエストが特定のいくつかのプロセスで処理され、他のプロセスは完全にアイドル状態になっていることがあります。

### プリエンプティブ機構
workermanの複数のプロセスによる接続取得は、**デフォルトで** **プリエンプティブ**な方法を取ります。つまり、クライアントが接続を確立すると、すべてのアイドル状態のプロセスがその接続を取得するためのチャンスを得ます。早い者勝ちです。誰が早いかは、オペレーティングシステムのカーネルのスケジューリングによって決まります。オペレーティングシステムは最近使用されたプロセスを優先してCPUを割り当てることがあります。なぜならCPUのレジスタには前回のプロセスのコンテキスト情報が残っている可能性があるからで、これによってコンテキストの切り替え負荷が減少します。そのため、ビジネスが非常に速い場合や、負荷テスト中には、リクエストが特定のプロセスに集中して処理されることがより頻繁に発生する可能性があります。なぜならこの戦略は、頻繁なプロセスの切り替えを避けることができ、性能が最適化されるからです。これは問題ではありません。

### ラウンドロビン機構
workermanは、`$worker->reusePort = true;`を設定することで、接続の取得方法を**ラウンドロビン**の方法に変更することができます。ラウンドロビンの方法では、カーネルは接続をほぼ均等にすべてのプロセスに割り当てるため、すべてのプロセスがリクエストを一緒に処理します。

### 誤解
多くの開発者は、全てのプロセスがリクエスト処理に参加すれば性能が向上すると考えていますが、実際には必ずしもそうとは限りません。ビジネスが非常にシンプルな場合、リクエスト処理に参加するプロセス数がCPUコア数に近づくほどサーバーのスループットが高くなります。たとえば、4つのコアを持つサーバーでは、プロセス数を4に設定すると、helloworldの負荷テストのQPSが通常最も高くなります。リクエスト処理のプロセス数がCPUコア数を大幅に超えると、プロセスのコンテキスト切り替えコストが増加し、パフォーマンスは逆に悪化する可能性があります。そして通常のデータベースを使用するビジネスでは、プロセス数をCPUコア数の3倍から6倍に設定すると、パフォーマンスが向上することがあります。
