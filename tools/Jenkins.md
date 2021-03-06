# Jenkins

<p align = "center">
    <a href="https://www.jenkins.io/" target="_Blank"><img src="https://img.shields.io/badge/Jenkins-v2.263.3 LTS-orange?logo=Jenkins" alt="Jenkins"/></a>
    <a href="https://maven.apache.org/" target="_Blank"><img src="https://img.shields.io/badge/Maven-v3.3.6-yellow?logo=Apache Maven" alt="Apache Maven"/></a>
    <a href="https://git-scm.com/" target="_Blank"><img src="https://img.shields.io/badge/Git-v2.29.0-pink?logo=Git" alt="Git"/></a>
    <a href="https://tomcat.apache.org/" target="_Blank"><img src="https://img.shields.io/badge/Tomcat-v9.0.39-purple?logo=Apache Tomcat" alt="Apache Tomcat"/></a>
    <a href="https://www.docker.com/" target="_Blank"><img src="https://img.shields.io/badge/Docker-v1.13.1-green?logo=Docker" alt="Docker"/></a>
    <a href="https://www.spring.io/" target="_Blank"><img src="https://img.shields.io/badge/Springboot-v2.3.4-blue?logo=spring" alt="spring"/></a>
</p>


## Introduction

### Jenkins是什么

Jenkins是一款开源 CI&CD 软件，用于自动化各种任务，包括构建、测试和部署软件。

Jenkins 支持各种运行方式，可通过系统包、Docker 或者通过一个独立的 Java 程序

官网地址：https://www.jenkins.io/

官网教程：https://www.jenkins.io/zh/doc

### Jenkins解决了什么问题

持续集成(CI) / 持续交付 (CD) 

项目部署到服务器这个过程 原本需要我们自己做的事情都有：

推送代码到版本控制器仓库--->使用maven clean清除原本的编译文件--->使用maven package打成指定的jar/war包--->将打好的包手动拷贝到服务器内的Servlet容器（Tomcat）--->重启Servlet容器

变为现在的：推送代码到版本控制器仓库--->到Jenkins确认一下--->项目完成更新

可如图所示：

![Jenkins-Flow](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/08/094003-710577.png)

## Getting Start

### 下载和安装

#### 下载

官网提供多种下载方式，可根据自己的情况如实下载和安装。

本节以通用 war 包的形式来下载和安装、还可通过 `DIS.md` 查看docker详情安装

war包官网下载地址：https://www.jenkins.io/download/

![image-20210208100051763](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/08/100055-164144.png)

在此页面下载指定版本的war文件即可

#### 安装

Jenkins的运行离不开Java环境的支持、而且依赖于Servlet容器，故介绍两种安装war的方式，可选其一使用：

- 将下载好的war文件放入 Servlet容器，这里以Tomcat为例，随后启动Tomcat后，可访问

```http
http://ip:port/jenkins/
```

来到指定Jenkins控制台

或

- 因Jenkins自带Servlet容器（Jetty）、内置容器后不需依赖外部容器也能运行，但依旧依赖于Java环境

```shell
java -jar jenkins.war
```

可使用如上命令运行指定war包

Jenkins默认启动端口为8080，如若端口被占用启动失败，可自定义启动端口

```shell
java -jar jenkins.war --httpPort=xxxx (1-65535)
```

启动后可访问

```http
http://ip:自定义端口或默认端口
```

##### 安装后向导

###### 解锁 Jenkins

当第一次访问新的Jenkins实例时，系统会要求使用自动生成的密码对其进行解锁

访问上述链接后可能会来到如下界面

![jenkins1-page](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/08/102317-970195.jpeg)

Windows系统的文件路径可能为 `C:/Users/用户名/.jenkins/secrets/initialAdminPassword`

如果使用docker安装，挂载文件处可以找到 `/secrets` 文件，向下找即可，如未挂载文件运行容器，则可使用进入 容器 `docker exec -it 容器id` 去指定目录查找，或访问 `docker logs -f`查看实时日志来寻找打印到控制台的密码

密码的前后通常伴随着 一串 `*` 号

可如图：

![setup-jenkins2](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/08/102327-994007.png)

将密码粘贴到框内后，点击继续

###### 自定义Jenkins插件

解锁 Jenkins之后，在 **Customize Jenkins** 页面内， 您可以安装任何数量的有用插件作为您初始步骤的一部分

插件对后期的集成部署有一定的帮助，如果你是刚刚接触Jenkins，我们建议你安装**社区推荐的插件集**

如果您不确定需要哪些插件，请选择 **安装建议的插件** 。 您可以通过Jenkins中的**Manage Jenkins** > **Manage Plugins** 页面在稍后的时间点安装（或删除）其他Jenkins插件

除此之外，你可能需要用到额外的几个插件，可依次下载：

- Maven Integration：可以在 **create-job** 处创建 Maven项目
- Deploy to container：可以打包后部署到容器
- Publish Over SSH：可以以 ssh 的方式推送给远程Servlet容器

###### 创建第一个管理员用户

最后，在customizing Jenkins with plugins之后，Jenkins要求您创建第一个管理员用户。 . 出现**创建第一个管理员用户**页面时， 请在各个字段中指定管理员用户的详细信息，然后单击 **保存完成** 当 **Jenkins准备好了** 出现时，单击*开始使用 Jenkins*。

如果不创建用户，选择使用 admin 方式继续访问，那么 密码和账号被默认更改为 `admin/默认生成密码` 可访问 **admin->settings->password** 修改密码

### Maven(web)

#### 前提说明

- 本节重要介绍基于`maven web`项目的自动化部署
- 本节介绍适用于所有`Linux`系统的`Java`程序，如果使用的是Windows,那么以下脚本处位置，需要更换为`Windows`脚本
- 本节所用技术为：`Maven(web)+Git+GitHub+Tomcat+Jenkins`
- 版本管理可依据文档头部描述

#### 环境准备

- Maven web项目的创建，并推送到Github指定仓库
- Jenkins的安装完成

#### 开始构建

以下步骤全部使用截图说明，需要额外说明时才会使用文字标注

- 启动Jenkins

来到初始页面

配置Jenkins全局设置

![image-20210210180552955](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/10/180602-558343.png)

![image-20210210180836567](C:\Users\chengjuan\AppData\Roaming\Typora\typora-user-images\image-20210210180836567.png)

> Maven

![image-20210210181208623](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/10/181209-336763.png)

![image-20210210180933575](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/10/180934-809089.png)

> JDK

![image-20210210181025868](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/10/181027-959909.png)

> Git

![image-20210210181056317](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/10/181056-536674.png)

返回主目录，点击创建工程

![image-20210210175729166](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/10/175730-787820.png)

![a](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/10/180308-694829.png)

来到如下界面

![image-20210210181337511](C:\Users\chengjuan\AppData\Roaming\Typora\typora-user-images\image-20210210181337511.png)

配置Git源码管理

![image-20210210181720215](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/10/181720-408644.png)

需要添加GitHub账户和密码，如下

![image-20210210181908492](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/10/181908-638352.png)

![image-20210210182327329](https://typora-i-1302727418.cos.ap-shanghai.myqcloud.com/typora/202102/10/182328-458505.png)

Linux就执行shell脚本，windows就执行批处理命令

这里拿Linux作为演示，选择执行shell脚本后，在文本框中输入脚本

