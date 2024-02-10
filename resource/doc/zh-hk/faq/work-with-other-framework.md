# 如何與其他框架整合
**問：**

如何與其他mvc框架（thinkPHP、Yii等）整合？

**答：**

![workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

與其他mvc框架結合**建議**以上圖的方式(以ThinkPHP為例)：

1、ThinkPHP與Workerman是兩個獨立的系統，獨立部署(可部署在不同伺服器)，互不干擾。

2、ThinkPHP以HTTP協議提供網頁頁面在瀏覽器渲染展示。

3、ThinkPHP提供的頁面的js發起websocket連接，連接Workerman。

4、連接後給Workerman發送一個數據包(包含使用者名稱密碼或者某種token串)用於驗證websocket連接屬於哪個使用者。

5、僅在ThinkPHP需要向瀏覽器推送數據時，才調用Workerman的socket接口推送數據。

6、其餘請求還是按照原本ThinkPHP的HTTP方式調用處理。


**總結：**

把Workerman作為一個可以向瀏覽器推送的通道，僅僅在需要向瀏覽器推送數據時才調用Workerman接口完成推送。業務邏輯全部在ThinkPHP中完成。


ThinkPHP如何調用Workerman socket接口推送數據參考[見常見問題-在其他項目中推送](push-in-other-project.md)一節

**ThinkPHP官方已經支援了workerman，參見[ThinkPHP5手冊](https://www.kancloud.cn/manual/thinkphp5/235128)**
