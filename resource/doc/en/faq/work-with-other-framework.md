# How to Integrate with Other Frameworks
**Question:**

How to integrate with other MVC frameworks (such as thinkPHP, Yii, etc.)?

**Answer:**

![workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

It is **suggested** to integrate with other MVC frameworks as shown in the above diagram (using ThinkPHP as an example):

1. ThinkPHP and Workerman are two independent systems, deployed independently (can be deployed on different servers), and do not interfere with each other.

2. ThinkPHP provides web pages to be rendered and displayed in the browser using the HTTP protocol.

3. The JavaScript provided by ThinkPHP's pages initiates a websocket connection to connect to Workerman.

4. After the connection is established, a data packet (including a username, password, or some kind of token string) is sent to Workerman to authenticate which user the websocket connection belongs to.

5. Workerman's socket interface is only called to push data to the browser when ThinkPHP needs to push data.

6. Other requests are still processed using the original HTTP method in ThinkPHP.

**Summary:**

Workerman serves as a channel for pushing data to the browser, and its socket interface is only called when it is necessary to push data to the browser. All business logic is handled within ThinkPHP.

For how ThinkPHP can call the Workerman socket interface to push data, refer to the section "Push in Other Projects" in the [FAQ](push-in-other-project.md).

**ThinkPHP officially supports Workerman, please refer to the [ThinkPHP 5 manual](https://www.kancloud.cn/manual/thinkphp5/235128)**
