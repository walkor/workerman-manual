# 安装
WorkerMan实际上没有安装脚本，如果你的PHP环境已经装好，只需要把WorkerMan源代码或者demo下载下来即可运行。

windows用户需要使用windows版本的workerman，windows版本不依赖任何扩展，只需要配置好php环境变量即可，windows版本及注意事项参见[**这里**](http://www.workerman.net/windows)。


可以使用以下脚本测试本机PHP环境是否满足WorkerMan运行要求。
```curl -Ss http://www.workerman.net/check.php | php```

上面脚本如果全部显示ok，则代表满足WorkerMan要求，直接到[官网](http://www.workerman.net/)下载例子即可运行。

如果缺少某个扩展，请参考[附录-安装扩展](http://doc3.workerman.net/appendices/install-extension.html)一节安装。

如果没有PHP环境，请根据系统参考以下步骤安装。

## 通过Github安装

### centos系统安装教程

1、命令行运行（此步骤包含了安装php-cli主程序以及pcntl、posix、libevent库及git程序）
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```


2、命令行运行（此步骤是通过pecl安装Event扩展。注意Event扩展不是必须的，开发环境或者并发连接数不超过1000的此步骤可以跳过）
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 时输入```no```回车，其它直接敲回车就行


3、命令行运行（此步骤是配置Event扩展的ini配置，如果不安装Event扩展此步骤跳过）
```shell
echo extension=event.so > /etc/php.d/event.ini
```


4、命令行运行（此步骤是通过github下载WorkerMan主程序）
```shell
git clone https://github.com/walkor/Workerman
```

5、参考[入门指引--简单开发实例部分](/getting-started/simple-example.html)写入口文件运行。<br>
或者从[官网](http://www.workerman.net/)下载打包好的demo运行。


### debian/ubuntu系统安装教程

1、命令行运行（此步骤包含了安装php-cli主程序、libevent库及git程序）
```shell
apt-get install php5-cli git gcc php-pear php5-dev libevent-dev -y
```


2、命令行运行（此步骤是通过pecl安装Event扩展。注意Event扩展不是必须的，开发环境或者并发连接数不超过1000的此步骤可以跳过）
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 时输入```no```回车，其它直接敲回车就行


3、命令行运行（此步骤是配置Event扩展的ini配置，如果不安装Event扩展此步骤跳过）
```shell
echo extension=event.so > /etc/php5/cli/conf.d/event.ini
```


4、命令行运行（此步骤是通过github下载WorkerMan主程序）
```shell
git clone https://github.com/walkor/Workerman
```

5、参考[入门指引--简单开发实例部分](/getting-started/simple-example.html)写入口文件运行。<br>
或者从[官网](http://www.workerman.net/)下载打包好的demo运行。

### mac os 系统安装教程
**方法1：**mac系统自带PHP Cli，但是可能缺少```pcntl```扩展。

1、参考手册[附录-安装扩展](http://doc3.workerman.net/appendices/install-extension.html)一节中方法三源码编译安装```pcntl```扩展。

2、参考手册[附录-安装扩展](http://doc3.workerman.net/appendices/install-extension.html)一节中方法四利用phpize安装```Event```扩展（可省略）。

3、通过http://www.workerman.net/download/workermanzip 下载WorkerMan主程序，或者到[官网](http://www.workerman.net/)下载例子运行。

**方法2：**通过```brew```命令安装php及对应扩展

1、命令行运行以下命令安装```brew```工具(如果已经安装过```brew```可以跳过此步骤)
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2、命令行运行以下命令安装```php7```
```
brew install php70
```

3、命令行运行以下命令安装```event```扩展
```
brew install php70-event
```

4、通过http://www.workerman.net/download/workermanzip 下载WorkerMan主程序，或者到[官网](http://www.workerman.net/)下载例子运行


## Event扩展说明
[Event扩展](http://php.net/manual/zh/book.event.php)不是必须的，当业务需要支撑上万并发连接时，推荐安装Event，能够支持巨大的并发连接。如果业务并发连接比较低，例如1000并发连接，则可以不用安装。

如果无法安装[Event扩展](http://php.net/manual/zh/book.event.php)，可以用[libevent扩展](http://php.net/manual/zh/book.libevent.php)代替，注意目前libevent扩展不支持php7，php7用户只能使用Event扩展。

**安装libevnet扩展方法如下：**

注意：libevnet扩展也同样依赖libevent库，所以首先需要安装libevent-devel包(并非扩展)。



centos系统

```
yum install libevent-devel
pecl install channel://pecl.php.net/libevent-0.1.0 //提示libevent installation [autodetect]: 时按回车
echo extension=libevent.so > /etc/php.d/libevent.ini
```

debian/ubuntu系统

```
apt-get install libevent-dev
pecl install channel://pecl.php.net/libevent-0.1.0 //提示libevent installation [autodetect]: 时按回车
echo extension=libevent.so > /etc/php5/cli/conf.d/libevent.ini
```


