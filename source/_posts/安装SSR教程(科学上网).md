---
title: 安装SSR教程（科学上网）
date: 2020-02-15 15:28:45
tags:
	- SSR
	- 科学上网
categories: SSR
---

# 安装SSR
## 获取安装脚本
<!-- more -->
```shell
$ wget --no-check-certificate https://github.com/zhanghanlun/ssr/raw/master/shadowsocksR.sh
```
## 授予脚本权限
```shell
$ chmod +x shadowsocksR.sh
```
## 安装ssr
```shell
$ ./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```
## ssr常用命令

```shell
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status
配置文件路径：/etc/shadowsocks.json
日志文件路径：/var/log/shadowsocks.log
代码安装目录：/usr/local/shadowsocks
```
## SSR多用户管理
打开配置文件shadowsocks.json，将配置文件文件修改为
```
{
    "server": "0.0.0.0",
    "port_password": {
        "8381": "foobar1",
        "8382": "foobar2",
        "8383": "foobar3",
        "8384": "foobar4"
    },
    "timeout": 300,
    "method": "aes-256-cfb"
}
```
重启ssr生效

# 安装bbr加速
## 获取bbr脚本
```shell
$ wget --no-check-certificate https://github.com/zhanghanlun/ssr/raw/master/bbr.sh
```
## 脚本赋予权限
```shell
$ chmod +x bbr.sh
```
## 执行脚本
```shell
$ /bbr.sh
```
判断是否安装成功
```shell
sysctl net.ipv4.tcp_available_congestion_control
```
返回的结果中有bbr则安装成功
```shell
net.ipv4.tcp_available_congestion_control = reno cubic bbr
```
# SSR客户端地址
将SSR安卓和win客户端上传到github了，链接如下
[下载地址](https://github.com/zhanghanlun/ssr)