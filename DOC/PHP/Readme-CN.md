## 入门指南

### 要求

依赖|版本
---|----
PHP| php 7+
GCC| GCC 5+
*inux|
Python | Python 3
pinpoint| 1.8.0-RC1
**Composer**| 

### 安装

#### 步骤
1. 执行此命令： git clone https://github.com/naver/pinpoint-c-agent.git
   
    > 搭建 pinpoint-php-module

2. 在您的pinpoint-c-agent安装包的根目录下，执行以下命令：
   1. phpize        
   2. ./configure
   3. make 
   4. make test TESTS=src/PHP/tests/ # 我们建议您执行此命令，测试该模块是否正确。
   5. make install 
   6. Activation pinpoint module # 将以下内容添加到 php/lib/php.ini 的配置文件中
   
       >  php.ini 
        ```ini
        extension=pinpoint_php.so
        # unix:(unix sock address)
        pinpoint_php.CollectorHost=unix:/tmp/collector-agent.sock
        # or TCP address
        #pinpoint_php.CollectorHost=Tcp:ip:port
        pinpoint_php.SendSpanTimeOutMs=0 # 0 is recommanded
        # request should be captured duing 1 second. < 0 means no limited
        pinpoint_php.TraceLimit=-1 
        ```
    > 搭建 Collector-agent
3. 在 collector-agent 目录下，执行以下命令：
   1. python3 -m venv env
   2. source env/bin/activate
   3. pip3 install -r requirements.txt
   4. 根据以下内容更改 conf/collector.conf 中的配置文件

        ```ini
        [Collector]
        # collectorhost and specific port
        ### 要使用thrift协议，pinpoint的版本需要在 1.8.0-RC1 以上
        #CollectorSpanIp=collectorHost
        #CollectorSpanPort=9905
        #CollectorStatIp=collectorHost
        #CollectorStatPort=9906
        #CollectorTcpIp=collectorHost
        #CollectorTcpPort=9907
        ### 要使用grpc协议，pinpoint的版本需要在 2.0 以上
        AgentID=your_id
        ApplicationName=your_name
        collector.grpc.agent.ip=collectorHost
        collector.grpc.agent.port=9991
        collector.grpc.stat.ip=collectorHost
        collector.grpc.stat.port=9992
        collector.grpc.span.ip=collectorHost
        collector.grpc.span.port=9993
        [Common]
        # your web server (nginx&apache) port
        Web_Port=8001   
        # debug in dev
        Log_Level=ERROR 
        # 确保 LOG_DIR 是存在的
        # 可以使用“tail -f”命令在此路径下查看collector-agent的实时日志
        LOG_DIR=/your log dir/ 
        [Agent]
        # the same as below "pinpoint_php.CollectorHost"
        # sock address
        Address=/tmp/collector-agent.sock
        # or TCP address
        #Address=ip@port
        ```
    5. export COLLECTOR_CONFIG=/full path of collector.conf/
    6. collector-agent的启动命令如下所示：
        ```
        $ ./init_python_env.sh
        $ python run.py 
        ```
   
    > 试试 aop example
4. 在 PHP/pinpoint_php_example 目录下，执行以下步骤：
   1. 下载 composer.phar. [如何使用 Composer?](https://getcomposer.org/doc/00-intro.md)
   2. 安装 php composer.phar 
   3. 将$PWD/app/设置为网站根路径 # 也就是将nginx的路径设置为配置文件conf/nginx.conf中的collection -agent路径。



### 待办事项


## 兼容性

Pinpoint 版本 | PHP 版本| Python 版本|GCC 版本
---------------- | ----- | -----|---
1.8.0-RC1 | PHP 7+ | Python3|GCC 5+

## 变化

- 完全支持 ZTS
- 更加稳定
- 性能更高 
- 易于使用和维护
- 支持 GRPC

## F & Q

### 1. 如何重新生成所有AOP文件？

 在Cache/中删除 *__class_index_table* 

### 2. 为什么不支持自动更新AOP文件?

我们可以支持但不建议这样做！因为我们还没有找到一种有效的方法来实现这一点，每次监测这些文件的状态结果都很糟糕。

### 3. 它在使用时损失了多少性能？

在完全测试之后，AOP代码可以在我们的环境(E5-2660 v4 @ 2.00GHz)中至少消耗1ms。但监测的功能/方法越多，损耗的性能也就越多。

[测试结果 ☚](./Readme-CN.md#压力测试下的性能损失)

[FLarum的测试结果 ☚](./User%20Manual-CN.md#1.1-性能测试结果)

### 对于开发人员

[插件编写指南 ☚](../../Example/PHP/Readme-CN.md)

[PHP用户手册 ☚](./User%20Manual-CN.md)

### 已知问题

#### 1. 带有错误文件路径的异常或错误消息。

https://github.com/naver/pinpoint-c-agent/issues/99

#### 2. 如果不使用composer/autoloader，我可以使用这个版本吗？

抱歉，php-simple-aop不支持打包没有composer/autoloader的用户类(或内部类)。顺便说一下，composer挺不错的O(∩_∩)O

https://github.com/naver/pinpoint-c-agent/issues/103

#### 3. 为什么不支持PHP5.x?

https://www.php.net/supported-versions.php

#### 4. ~7.4 不支持generator函数。~ 此问题已解决!

https://github.com/naver/pinpoint-c-agent/issues/100

#### 5. 一些内置函数不支持AOP。
https://github.com/naver/pinpoint-c-agent/issues/102


## v0.1.x 和 v0.2.x 的区别

##  v0.1.x VS. v0.2.x

> 为什么要重构pinpoint_c_agent (0.1.x)？

1. 很难支持所有php版本
2. 速度太慢, v0.1.x 挂钩所有函数
3. 不支持ZTS

### 数据线路图

#### v0.1.x

> AOP 基于 zend_excuted_hook

![How does it work](../../images/principle_v0.1.x.png)
#### v0.2.x

> AOP 基于 classloader (比如 java-classloaders)

![How does it work](../../images/principle_v0.2.x.png)

#### PHP 版本兼容性

Agent 版本|PHP5.5.x|PHP5.6.x|PHP7.x |php-zts
----|-----|----|-----|---
v0.1.x|✔|✔|✔|✘
v0.2.x|✘|✘|✔|✔

#### PHP 框架

框架|v0.1.x|v0.2.x
----|-----|----|
Laravel|✔|✔
ThinkPHP|✔|✘
YII|✔|✔
Workerman|✘|✔
EasySwoole|✘|✔

> 备注

1. 如果您的php应用程序不支持composer(如woredpress、phpwind等)，那么就只能使用v0.1.x。

#### 稳定性

`v0.2.x > v0.1.x`


#### 可维护性(动态地)

```
✔: 不会阻塞用户的应用程序
✘: 阻塞用户的应用程序: 应该重启php-fpm/apache 
```

操作|v0.1.x|v0.2.x
----|-----|----
1.更新插件(CRUD) |✘|✔ [如何更新?](https://github.com/eeliu/php_simple_aop#how-to-reload-all-plugins)
2.更新 pinpoint collector|✘|✔
3.更新 pinpoint_php.so(pinpoint.so)|✘|✘



#### 压力测试下的性能损失

> 测试场景

```
测试对象: PHPMYAdmin
网址: http://dev-mysql-phpmyadmin/db_structure.php?server=1&db=testDB
插件数量: 4
Hooked 次数: 30
```

> 测试结果

案例|MTT
---|----
phpmyadmin without pinpoint|387.28ms
phpmyadmin with pinpoint-php v0.1|511.46ms
phpmyadmin with pinpoint-php v0.2|398.26ms


```
MTT: Mean RTT Time 
```
> [ pinpoint-php on Flraum 测试结果 ](./User%20Manual-CN.md#1.1-性能测试结果)

> 顺便提一下，Pinpoint是一个APM系统，但不是一个性能增强系统。目前这种损失是无法避免的

### 编者的话

由于人们广泛使用composer，所以v0.2.x是我们长期支持的版本。v0.1.x在未来可能会停止更新和维护，目前我们会修复v0.1.x中的一些致命错误。