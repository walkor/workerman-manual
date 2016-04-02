# 安装
WorkerMan实际上没有安装脚本，如果你的PHP环境已经装好，只需要把WorkerMan源代码或者demo下载下来即可运行。


可以使用以下脚本测试本机PHP环境是否满足WorkerMan运行要求。
```curl -Ss http://www.workerman.net/check.php | php```

上面脚本如果全部显示ok，则代表满足WorkerMan要求，直接到[官网](http://www.workerman.net/)下载例子即可运行。

如果缺少某个扩展，请参考[附录-安装扩展](http://doc3.workerman.net/appendices/install-extension.html)一节安装。

如果没有PHP环境，请根据系统参考以下步骤安装。

## 通过Github安装

### centos系统安装教程

1、命令行运行（此步骤包含了安装php-cli主程序以及pcntl、posix、libevent扩展及github程序）
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel
```


2、命令行运行（此步骤是通过pecl安装libevent扩展，如果失败请尝试按照安装扩展 一节中使用源码phpize的方式安装。注意libevent扩展不是必须的，此步骤可以跳过）
```shell
pecl install channel://pecl.php.net/libevent-0.1.0
```


3、命令行运行（此步骤是配置libevent扩展的ini配置，如果不安装libevent此步骤跳过）
```shell
echo extension=libevent.so > /etc/php.d/libevent.ini
```
注意在提示libevent installation [autodetect]: 时按回车即可


4、命令行运行（此步骤是通过github下载WorkerMan主程序）
```shell
git clone https://github.com/walkor/Workerman
```

### debian/ubuntu系统安装教程

1、命令行运行（此步骤包含了安装php-cli主程序、libevent扩展及github程序）
```shell
apt-get install php5-cli git gcc php-pear php5-dev libevent-dev
```


2、命令行运行（此步骤是通过pecl安装libevent扩展，如果失败请尝试按照 4.1 环境要求 一节中使用源码phpize的方式安装。注意libevent扩展不是必须的，此步骤可以跳过）
```shell
pecl install channel://pecl.php.net/libevent-0.1.0
```
　  提示libevent installation [autodetect]: 时按回车


3、命令行运行（此步骤是配置libevent扩展的ini配置，写这个配置需要切换到root用户，如果不安装libevent此步骤跳过）
```shell
echo extension=libevent.so > /etc/php5/cli/conf.d/libevent.ini
```


4、命令行运行（此步骤是通过github下载WorkerMan主程序）
```shell
git clone https://github.com/walkor/Workerman
```

### mac os 系统安装教程
mac系统自带PHP，但是可能缺少pcntl扩展。

1、参考手册[附录-安装扩展](http://doc3.workerman.net/appendices/install-extension.html)一节中方法三源码编译安装pcntl扩展。

2、参考手册[附录-安装扩展](http://doc3.workerman.net/appendices/install-extension.html)一节中方法四利用phpize安装libevent扩展（可省略）。

3、通过http://www.workerman.net/download/workermanzip 下载WorkerMan主程序，或者到[官网](http://www.workerman.net/)下载例子运行。


## libevent扩展说明
[libevent扩展](http://php.net/manual/zh/book.libevent.php)不是必须的，当业务需要支撑上万并发连接时，推荐安装libevent，能够支持巨大的并发连接。如果业务并发连接比较低，例如5000并发连接，则可以不用安装。

如果无法安装[libevent扩展](http://php.net/manual/zh/book.libevent.php)，可以用[Event扩展](http://php.net/manual/zh/book.event.php)代替。

**安装Event扩展方法如下：**

注意：Event扩展也同样依赖libevent库，所以首先需要安装libevent-devel包(并非扩展)。

centos系统

```
yum install libevent-devel
pecl install event
echo extension=event.so > /etc/php.d/event.ini
```

debian/ubuntu系统

```
apt-get install libevent-dev
pecl install event
echo extension=event.so > /etc/php5/cli/conf.d/event.ini
```


