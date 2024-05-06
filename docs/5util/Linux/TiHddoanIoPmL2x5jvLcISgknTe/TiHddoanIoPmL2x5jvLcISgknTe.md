# Java 环境安装和 Java 项目部署

# 前言

身为一个 Java 开发程序员，Java 环境的安装是基本项。开发环境本身在 Windows 系统上安装，其基本的流程已经非常熟悉，在此不做赘述

本篇主要记录一下，为了在 Linux 系统上部署 Java 项目的流程，便于未来复用

- 注意，安装 Java 的版本需要与部署项目的 Java 版本一致。比如我这次，就先安装了 Java1.8，结果发现无法使用，然后卸载掉重新安装了 Java17，才成功运行

> 系统环境：
> CentOS Linux release 7.9.2009 (Core)

# Java 安装与环境配置

## 卸载 Java

- 卸载 Java 的 JDK
  ```bash
  rpm -qa | grep -i java # 查询 Java JDK 信息
  rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps # 全部移除，或按需移除
  ```

- 删除环境变量，翻到最后，找到类似`JAVA_HOME`、`PATH`等行并删除
	```bash
  vim /etc/profile
  source /etc/profile # 立即生效
  ```

## yum 安装 JDK

```bash
yum -y list java* # 查询可安装的Java
yum install -y java-1.8.0-openjdk.x86_64 # 安装
java -version # 安装成功
```

## 自定义安装 Java

> Oracle 官网：https://www.oracle.com/cn/java/technologies/downloads/

- 前往官网下载 Java 安装包，记得选择需要的版本中的 Linux 系统对应啊安装包
- 建立安装目录：`mkdir /usr/local/software/java`
- 下载并解压

  ```bash
  wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz
  tar -zxvf ./jdk-17_linux-x64_bin.tar.gz
  ```

- 添加环境变量
	- 在`/etc/profile`底部添加如下内容，我的JDK安装路径
  
	```
  JAVA_HOME=/usr/local/software/java/jdk-17.0.11
  PATH=$PATH:$JAVA_HOME/bin
  export JAVA_HOME PATH
  ```
  ```bash
  vim /etc/profile
  source /etc/profile # 理解生效
  java --version # 安装成功
  ```

# 项目部署

本次项目使用Spring boot开发，无后端无数据库，因此直接后台运行后端项目即可。

最初测试采用的是直接`java -jar [xxx].jar`进行运行和测试的，但是在关闭XShell之后，第二天进入发现程序又退出了。因此采用`nohup`后台运行程序，这样可以在服务器上一直运行

```bash
nohup java -jar [xxxx].jar # nohub表示后台运行
```

部署成功！