# Workermanはマルチスレッドをサポートしていますか？

Workermanには[pthreads拡張](https://php.net/manual/zh/book.pthreads.php)に依存する[MTマルチスレッドバージョン](https://github.com/walkor/workerman-MT)がありますが、pthreads拡張がまだ安定していないため、このWorkermanマルチスレッドバージョンはもはやメンテナンスされていません。

**現在、Workermanおよび関連製品はすべてマルチプロセス単一スレッドに基づいています。**
