# 安装说明
WorkerMan实际上就是一个PHP代码包，如果你的PHP环境已经装好，只需要把WorkerMan源代码或者demo下载下来即可运行。

# windows用户（必读）

windows用户需要使用windows版本的workerman，windows版本workerman本身**不依赖任何扩展**，只需要配置好PHP环境变量即可，**windows版本workerman安装及注意事项参见[windows用户必看](http://www.workerman.net/windows)。**

**本页面以下本内容不适用于windows版本workerman，windows用户请忽略。**


# 环境检测
可以使用以下脚本测试本机PHP环境是否满足WorkerMan运行要求。
```curl -Ss http://www.workerman.net/check.php | php```

上面脚本如果全部显示ok，则代表满足WorkerMan要求，直接到[官网](http://www.workerman.net/)下载例子即可运行。

（注意：检测脚本中没有检测event扩展或者libevent扩展，如果业务并发连接数大于1024建议安装event扩展或者libevent扩展，安装方法参照下面说明）

# 已有PHP环境安装缺失扩展

## 安装pcntl和posix扩展：

**centos系统**<br>
如果php是通过yum安装的，则命令行运行 ```yum install php-process```即可安装pcntl和posix扩展。


如果安装失败或者php本身不是用yum安装的请参考手册[附录-安装扩展](http://doc3.workerman.net/appendices/install-extension.html)一节中方法三源码编译安装。

**debian/ubuntu/mac os系统**<br>
参考手册[附录-安装扩展](http://doc3.workerman.net/appendices/install-extension.html)一节中方法三源码编译安装。


## 安装event或者libevent扩展：
为了能支持更大的并发连接数，建议安装event扩展或者libevent扩展(二者作用相同，二选一即可)。以Event为例，安装方法如下:

**centos系统安装**

1、命令行运行（安装event扩展依赖的libevent-devel包）
```shell
yum install libevent-devel -y
```

2、命令行运行（安装event扩展）
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 时输入```no```回车，其它直接敲回车就行

如果安装失败请跳过以下步骤，尝试安装libevent扩展，见本页面底部。

3、命令行运行（如果ini文件位置不对，可以通过运行```php --ini```找到实际加载的ini文件路径）
```shell
echo extension=event.so > /etc/php.d/event.ini
```

**debian/ubuntu系统安装**

1、命令行运行（安装event扩展依赖的libevent-dev包）
```shell
apt-get libevent-dev -y
```

2、命令行运行（安装event扩展）
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 时输入```no```回车，其它直接敲回车就行

如果安装失败请跳过以下步骤，尝试安装libevent扩展，见本页面底部。

3、命令行运行(需要切换到root用户。如果ini文件位置不对，可以通过运行```php --ini```找到实际加载的ini文件路径)
```shell
echo extension=event.so > /etc/php5/cli/conf.d/event.ini
```

### mac os 系统安装教程

mac 系统一般作为开发机，不必安装event扩展。


## 全新系统安装（全新安装PHP+扩展）

### centos系统安装教程

1、命令行运行（此步骤包含了安装php-cli主程序以及pcntl、posix、libevent库及git程序）
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```


2、命令行运行（安装event扩展）
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 时输入```no```回车，其它直接敲回车就行

如果安装失败请跳过以下步骤3，尝试安装libevent扩展，见本页面底部。


3、命令行运行（此步骤是配置Event扩展的ini配置，如果ini文件位置不对，可以通过运行```php --ini```找到实际加载的ini文件路径）
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


2、命令行运行（安装event扩展）
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 时输入```no```回车，其它直接敲回车就行

如果安装失败请跳过以下步骤3，尝试安装libevent扩展，见本页面底部。


3、命令行运行（需要切换到root用户。此步骤是配置Event扩展的ini配置，如果ini文件位置不对，可以通过运行```php --ini```找到实际加载的ini文件路径）
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


