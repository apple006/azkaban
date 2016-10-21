Azkaban 3 [![Build Status](http://img.shields.io/travis/azkaban/azkaban.svg?style=flat)](https://travis-ci.org/azkaban/azkaban)
========

从源代码编译
--------------------
azkaban使用gradle编译，如果你当前环境下没有gradle，你可以到[github.com/gradle/gradle](github.com/gradle/gradle)下载源码编译或者下载发布的二进制包

从源代码编译Azkaban，执行下面命令：

```
./gradlew distTar
```

上面的命令把所有的Azkaban下的包打包为.zip.tar格式，如果在window下，你可以执行下面的命令打包为zip格式：

```
./gradlew distZip
```

If not building for the first time, it's good to clean first:

如果不是第一次编译，你最好clean一下：

```
./gradlew clean
```

文档
-------------

Azkaban文档, 请去[Azkaban Project Site](http://azkaban.github.io). 文档的源码在 [gh-pages branch](https://github.com/azkaban/azkaban/tree/gh-pages)可以找到

需要帮助的，请到Azkaban Google小组: [Azkaban Group](https://groups.google.com/forum/?fromgroups#!forum/azkaban-dev)

以下根据azkaban文档翻译，[文档地址](http://azkaban.github.io/azkaban/docs/latest/#solo-setup)

介绍
------------
Azkaban是一个在LinkedIn开发用来跑Hadoop作业的一款批处理作业调度框架，Azkaban通过作业的依赖关系解决了作业顺序和提供了更加方便的Web用户界面来维持和跟踪你的作业

特性
------------
* 兼容所有版本的 Hadoop
* 基于 Web 的易用 UI
* 简单的 Web 和 HTTP 工作流上传
* 项目工作空间
* 工作流调度
* 模块化和插件化
* 支持认证和授权
* 可跟踪用户行为
* 失败和成功时的邮件提醒
* SLA 警告和自动终止
* 失败作业的重试

概述
------------
Azkaban是由LinkedIn实现的为了解决hadoop作业依赖问题一个批处理调度平台。如果需要我们的作业按照顺序执行，从作业中ETL（抽取extract、转换transform、加载load）数据来分析产品，那么Azkaban将是一个很好的选择。

随着hadoop用户数量的增长，从最初的单个服务器的解决方案，Azkaban已经演变成更加健壮的解决方案。

Azkaban包含了3个关键的组件

* 关系型数据库服务器 (MySQL)
* Azkaban Web服务器 （AzkabanWebServer）
* Azkaban 执行服务器 （AzkabanExecutorServer）

![](https://github.com/silence940109/azkaban/blob/master/image/azkaban2overviewdesign.png)

###关系型数据库（MySQL）
Azkaban使用MySQL来存储他本身的状态信息，同样的AzkabanWebServer
和AzkabanExecutorServer也使用数据库。

####AzkabanWebServer怎么要使用数据库？
web服务器使用数据库有如下原因：

* 项目管理--项目的权限以及文件上传等
* 执行状态--对执行器正在执行的作业流进行跟踪
* 预览工作流--查找以前执行过的作业流以及访问它们的日志文件
* 调度器--保持被调度执行的作业的状态
* SLA(服务品质协议)--保存所有的SLA规则信息

AzkabanExecutorServer为什么要使用数据库？

* 访问工程--在数据库中检索项目文件
* 执行流/作业--为了正在执行的流检索和更新数据
* 日志--为作业和流往数据库中存储和输出日志
* 交互依赖--如果作业流运行在不同的执行器上，它会从数据库中读取状态信息

这里选择使用MySQL作为数据库是没有多少原因的，因为MySQL是一个广泛使用的数据库。我们也希望实现与其他类型数据库兼容，但是在搜索历史运行作业的需求上关系型数据库还是一个很好的选择。

####AzkabanWebServer

AzkabanWebServer是Azkaban最主要的管理者，它解决了项目的管理，认证，调度以及执行器的调度。它同样的也作为一个用户接口服务。

使用Azkaban是相当容易的，Azkaban使用 `*.job` 键-值属性文件来定义在一个工作流中的单个任务，使用 `_dependencies_`来定义作业链之间的依赖关系。这些作业文件和关系代码可以被压缩成 `.zip` 文件然后通过Azkaban用户界面或者curl上传到Web服务器。

####AzkabanExecutorServer

Azkaban之前的版本在同一台服务器上有AzkabanWebServer和AzkabanExecutorServer的特性，后台执行器被分开到它自己的服务器上。分离这些服务是有些原因的：我们可以快速的测量执行程序的规模以及当执行器执行失败时可以进行回滚，同样的，我们能够以最小对用户的影响来更新Azkaban。随着Azkaban使用量的增加，我们发现升级Azkaban变得越来越困难，因为所有的时间服务器都变得“高峰”。

开始
------------
在3.0的版本中我们提供了三种模式：单例“solo-server”模式，重量级的双服务器模式和分布式多执行器模式。以下描述这几种模式之间的不同。

在Solo服务器模式中，数据库是使用H2，并且web服务器和执行器服务器运行在同一个进程当中，如果只是其中一个服务器想做某些事，这是非常有用的。它同样的夜可以使用在小规模的测试用例。

##1.下载和安装Solo服务器安装包

**构建AzkabanSolo服务器**

在Azkaban2.5中有一种solo服务器模式来让你尝试使用Azkaban或者在一些小规模和安全性低的环境下使用。

它的特性有：

* 简单安装 - MySQL不是必须的，它内置了H2数据库来作为主要的实例存储
* 方便启动 - web服务器和执行器服务器运行在同一个进程中
* 丰富的特性 - 它包含Azkaban所有的特性，你可以正常使用它或者为它安装插件

**安装Solo服务器**

可以从[下载页](https://azkaban.github.io/downloads.html****)获取azkaban-exec-server安装包.

你也可以克隆[Github仓库](https://github.com/azkaban/azkaban2)，你可以从主分支上编译最新的版本，并且完成从源代码上编译.

从安装包上解压到目录下，解压后，里面应该有如下的目录：


	文件夹                    描述
    bin                      启动Azkaban的jerry服务器脚本
    conf                     Azkaban solo服务器配置信息
    lib                      Azkaban依赖的jar包
    extlib                   额外的jar包，这些jar包将会被添加到Azkaban的类路径下
    plugins                  插件安装的目录
    web                      Azkaban web服务器的web(css,javascript,image)文件

在`conf`目录中必须有三个文件：

* azkaban.private.properties - Azkaba运行时参数信息
* azkaban.properties - Azkaba运行时参数信息
* global.properties - 工具静态属性，用于给每个工作流和作业分享属性信息
* azkaban-users.xml - 用来添加用户和角色权限，如果XmLUserManager如果没有构建的话这个文件是没有用的

azkaban.properties文件是主要的配置文件

###为SSL获取秘钥
Azkaban solo服务器默认是不使用SSL协议的，但是你可以以同样的方式建立在单个服务器上，这是为什么：

Azkaban web服务器可以使用SSL套接字建立连接，这意味着秘钥是可以得到的。你可以按照这个连接([http://docs.codehaus.org/display/JETTY/How+to+configure+SSL](http://docs.codehaus.org/display/JETTY/How+to+configure+SSL))的步骤创建一个，一旦秘钥文件被创建，Azkaban必须给出它 azkaban.properties位置和密码等信息，下面的属性应该要被重写：

	jetty.keystore=keystore
	jetty.password=password
	jetty.keypassword=password
	jetty.truststore=keystore
	jetty.trustpassword=password

###用户管理配置
Azkaba使用用户管理来提供权限和用户角色，默认的，Azkaban包含和使用XmlUserManager从`azkaban-users.xml`文件中获取用户名和密码，这你可以在`azkaban.properties`文件中看到

* user.manager.class=azkaban,user.XmlUserManager
* user.manager.xml.file=conf/azkaban-user.xml

###启动Web服务器
下面的属性是在 `azkaban.properties文件中`用来配置jetty服务器的配置信息
	
	jetty.maxThreads=25
	jetty.ssl.port=8081
执行 bin/azkaban-solo-start.sh来启动solo服务器，如果要关闭，执行bin/azkaban-solo-shutdown.sh

在你的浏览器中打开[http://localhost:8081/index](http://localhost:8081/index)连接
      
##2.安装Azkabn插件*

