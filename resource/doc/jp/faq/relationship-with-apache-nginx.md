# ApacheおよびNginxとの関係
**質問:**

WorkermanとApache/nginx/php-fpmの関係は何ですか？WorkermanとApache/nginx/php-fpmは競合しますか？

**回答:**
WorkermanとApache/nginx/php-fpmには何の関係もなく、また、Workermanの運用はApache/nginx/php-fpmに依存していません。それらはそれぞれ独立したコンテナであり、お互いに干渉せず、また（同じポートを監視していない限り）競合することはありません。

Workermanは汎用のソケットサーバーフレームワークであり、長期間の接続をサポートし、HTTP、WebSocket、およびカスタムプロトコルなど、さまざまなプロトコルをサポートしています。一方、Apache/nginx/php-fpmは一般的にHTTPプロトコルのWebプロジェクトの開発に使用されます。

サーバーにすでにApache/nginx/php-fpmが展開されている場合、Workermanを展開してもそれらの運用には影響しません。
