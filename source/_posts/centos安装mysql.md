---
title: Centos7安装MySQL
date: 2018-02-04 17:24:45
tags:
	- mysql
categories: mysql
---

## 获取mysql安装包

<!-- more -->

```shell
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

## 安装mysql
```shell
yum install mysql mysql-server
```
## 数据库启动，并访问
```sql
systemctl start mysqld
mysql -u root
```

## 授权远程访问
```sql
use mysql;
grant all privileges  on *.* to root@'%' identified by "root";
FLUSH PRIVILEGES;
```

## 修改登录密码
```sql
use mysql;  
update user set password=password('123') where user='root';  
flush privileges;  
```