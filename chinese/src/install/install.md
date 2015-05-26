# 安装
WorkerMan实际上没有安装脚本，如果你的php环境已经装好，只需要把WorkerMan源代码下载下来即可运行。
下载方法建议用Github clone，你也可以通过下载zip文件的方式下载WorkerMan代码程序。

## 通过Github安装

### centos系统安装教程

1、命令行运行（此步骤包含了安装php-cli主程序以及pcntl、posix、libevent扩展及github程序）
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel
```


2、命令行运行（此步骤是通过pecl安装libevent扩展，如果失败请尝试按照 10.2 安装扩展 一节中使用源码phpize的方式安装。注意libevent扩展不是必须的，此步骤可以跳过）
```shell
pecl install channel://pecl.php.net/libevent-0.1.0
```


3、命令行运行（此步骤是配置libevent扩咱的ini配置，如果不安装libevent此步骤跳过）
```shell
echo extension=libevent.so > /etc/php.d/libevent.ini
```
注意在提示libevent installation [autodetect]: 时按回车即可


4、命令行运行（此步骤是通过github下载WorkerMan主程序）
```shell
git clone https://github.com/walkor/workerman
```



### debian/ubuntu系统安装教程(如果不是root用户请用sudo 后面加命令)

1、命令行运行（此步骤包含了安装php-cli主程序、libevent扩展及github程序）
```shell
apt-get install php5-cli git gcc php-pear php5-dev libevent-dev
```


2、命令行运行（此步骤是通过pecl安装libevent扩展，如果失败请尝试按照 4.1 环境要求 一节中使用源码phpize的方式安装。注意libevent扩展不是必须的，此步骤可以跳过）
```shell
pecl install channel://pecl.php.net/libevent-0.1.0
```
　  提示libevent installation [autodetect]: 时按回车


3、命令行运行（此步骤是配置libevent扩展的ini配置，如果不安装libevent此步骤跳过）
```shell
echo extension=libevent.so > /etc/php5/cli/conf.d/libevent.ini
```


4、命令行运行（此步骤是通过github下载WorkerMan主程序）
```shell
git clone https://github.com/walkor/workerman
```


##下载ZIP文件安装


1、前提条件你本地安装了必要的运行环境，安装方法根据你的系统参考上面1-3步骤

2、通过http://www.workerman.net/download/workermanzip 连接下载WorkerMan


