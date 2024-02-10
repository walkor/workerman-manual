# 他のフレームワークとの統合方法
**質問:**

他のMVCフレームワーク（thinkPHP、Yiiなど）とどのように統合できますか？

**回答:**

![workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

他のMVCフレームワークとの統合は、上記の方法（ThinkPHPを例に取る）が**推奨**されます：

1. ThinkPHPとWorkermanは独立したシステムであり、独自に展開されます（異なるサーバーに展開することも可能）。お互いに干渉しません。

2. ThinkPHPはHTTPプロトコルを使用してブラウザでページをレンダリング・表示します。

3. ThinkPHPが提供するページのJavaScriptは、WebSocket接続を作成し、Workermanに接続します。

4. 接続後、Workermanにはユーザーを認証するためのデータパケット（ユーザー名とパスワード、またはある種のトークン文字列）が送信されます。

5. ブラウザにデータをプッシュする必要がある場合にのみ、ThinkPHPはWorkermanのソケットインターフェースを呼び出してデータをプッシュします。

6. その他のリクエストは、元のThinkPHPのHTTP方法で処理されます。

**まとめ:**

Workermanをブラウザにデータをプッシュできるチャネルとして使用し、ブラウザにデータをプッシュする必要があるときにのみWorkermanのインターフェースを呼び出します。ビジネスロジックはすべてThinkPHPで完了します。

ThinkPHPからWorkermanのソケットインターフェースを呼び出してデータをブラウザにプッシュする方法については、[よくある質問-他のプロジェクトでのプッシュ](push-in-other-project.md)セクションを参照してください。

**ThinkPHP公式はWorkermanをすでにサポートしており、[ThinkPHP5マニュアル](https://www.kancloud.cn/manual/thinkphp5/235128)を参照してください**
