# 如何与其它框架整合
**问：**

如何与其它mvc框架（thinkPHP、Yii等）整合？

**答：**

![workerman-thinkphp](http://www.workerman.net/img/doc/workerman-work-with-thinkphp.png)

与其它mvc框架结合建议以上图的方式(ThinkPHP为例)：

1、ThinkPHP与Workerman是两个独立的系统，独立部署(可部署在不同服务器)，互不干扰。

2、ThinkPHP以HTTP协议提供网页页面在浏览器渲染展示。

3、ThinkPHP提供的页面的js发起websocket连接，连接workerman

4、连接后给Workerman发送一个数据包(包含用户名密码或者某种token串)用于验证websocket连接属于哪个用户。

5、仅在ThinkPHP需要向浏览器推送数据时，才调用workerman的socket接口推送数据。

6、其余请求还是按照原本ThinkPHP的HTTP方式调用处理。


**总结：**

把Workerman作为一个可以向浏览器推送的通道，仅仅在需要向浏览器推送数据时才调用Workerman接口完成推送。业务逻辑全部在ThinkPHP中完成。


ThinkPHP如何调用Workerman socket接口推送数据参考[见常见问题-在其它项目中推送](/faq/push-in-other-project.html)一节

