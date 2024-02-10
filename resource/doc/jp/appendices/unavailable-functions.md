# サポートされていない関数

サポートされていない関数/ステートメント  | 代替手段 | 説明
----|------|----
pcntl_fork | 事前にプロセス数を設定する | 
php://input | [`$request->rawBody()`](http/request.md) | HTTPプロトコルを使用してPOSTデータの元のデータを取得するために使用
exit | return | exitを使用するとプロセスが終了するため、返す場合はreturn文を直接使用してください
die | return | dieを使用するとプロセスが終了するため、返す場合はreturn文を直接使用してください
header cookie session関連の関数 | [`$request`](http/request.md) や [`$response`](http/response.md) クラスを参照してください | 
set_time_limit | なし | 0にのみ設定できます。それ以外の値を設定すると、一定時間後にworkermanプロセスが終了します
