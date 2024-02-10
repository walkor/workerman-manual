## Linuxシステムでworkermanを起動時に自動実行する方法

/etc/rc.localを開き、以下のようなコードを```exit 0```の前に追加してください。

```bash
ulimit -HSn 102400
/usr/bin/env php /ディスク/パス/start.php start -d

exit 0
```
