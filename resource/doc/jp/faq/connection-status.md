# 現在のクライアント接続数を確認する
```php start.php status```を実行すると、現在のサーバーのWorkermanの実行状況が確認できます。```connections```フィールドは各プロセスの現在のTCP接続数を示しています。注意すべき点は、このフィールドにはクライアントのTCP接続数だけでなく、Workerman内部の通信のTCP接続数も含まれているということです。たとえば、WorkermanのGateway/Workerモデルでは、各Gatewayプロセスの現在のクライアント接続数は```connections```フィールドの値からWorkerプロセスの数を引いたものになります。
