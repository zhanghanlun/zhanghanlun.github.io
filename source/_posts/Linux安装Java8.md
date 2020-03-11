---
title: Linux安装Java8
tags:
  - Linux
  - Java
categories: Linux
abbrlink: 1161758996
date: 2020-03-11 14:51:45
---
# 下载Java8 tar.gz包
上[Java官网](https://www.oracle.com/java/technologies/javase-jdk8-downloads.html)
<!-- more -->
![在这里插入图片描述](https://upyun.zhanghanlun.com/blog/2020/03/2020_311114456.jpg)
下载Java压缩包
# 解压并配置路径
创建目录
```shell
mkdir -p /usr/local/java
```
解压到目标文件夹
```
tar -vzxf jdk-8u241-linux-x64.tar.gz -C /usr/local/java/
```
# 添加环境变量
```shell
vim /etc/profile
```
添加入下内容，并保存
```shell
export JAVA_HOME=/usr/local/java/jdk1.8.0_241
export CLASSPATH=$:CLASSPATH:$JAVA_HOME/lib/
export PATH=$PATH:$JAVA_HOME/bin
```
重新加载配置文件
```
source /etc/profile
```
# 测试
```shell
[root@izj6c37yicmwutvw4fvvzxz ~]# java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```