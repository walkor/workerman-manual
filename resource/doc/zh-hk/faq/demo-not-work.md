# workerman例子無法運作
## 現象
workerman已經正常啟動，但按官網的範例或下載的示例無法運作，例如無法打開頁面，socket連接失敗等。

## 解決方法
一般這種workerman啟動無報錯，但無法打開頁面或無法連接的問題是由於伺服器防火牆導致。請先關閉伺服器防火牆再進行測試，如果確認是防火牆問題，請重新設定防火牆規則。
