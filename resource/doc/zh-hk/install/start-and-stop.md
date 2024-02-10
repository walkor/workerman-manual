# 啟動與停止

請注意，Workerman 的啟動和停止命令均在命令行中完成。

要啟動 Workerman，首先需要一個啟動入口文件，其中定義了服務監聽的端口及協議。可以參考[入門指引--簡單開發實例部分](../getting-started/simple-example.md)

以 [workerman-chat](https://www.workerman.net/workerman-chat) 為例，它的啟動入口為 start.php。

### 啟動

以調試（debug）方式啟動

```php start.php start```

以守護進程（daemon）方式啟動

```php start.php start -d```

### 停止

```php start.php stop```

### 重啟

```php start.php restart```

### 平滑重啟

```php start.php reload```

### 查看狀態

```php start.php status```

### 查看連接狀態（需要Workerman版本>=3.5.0）

```php start.php connections```

## 調試和守護進程方式區別

1、以調試方式啟動，代碼中的 echo、var_dump、print 等打印函數會直接輸出在終端。

2、以守護進程方式啟動，代碼中的 echo、var_dump、print 等打印會默認重定向到 /dev/null 文件，可以通過設定```Worker::$stdoutFile = '/your/path/file';```來設置這個文件路徑。

3、以調試方式啟動，終端關閉後，Workerman 會隨之關閉並退出。

4、以守護進程方式啟動，終端關閉後，Workerman 繼續在後台正常運行。

## 什麼是平滑重啟？

請參見 [平滑重啟原理](../faq/reload-principle.md)
