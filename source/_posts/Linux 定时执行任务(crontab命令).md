---
title: Linux 定时执行任务(crontab命令)
tags:
  - Linux
  - 定时任务
categories: Linux
abbrlink: 7c68ca2e
date: 2019-06-12 17:24:45
---
# 	1、前言
&ensp;&ensp;在Linux中我们经常会需要定时去执行一些任务脚本。在Linux中有这样一个命令就是帮助我们定时执行任务脚本的。它就是**crontab**。cron是一个linux下的定时执行工具，可以在无需人工干预的情况下运行作业。
<!-- more -->

#  2、crontab命令
写定时任务
```shell
#写定时任务
crontab -e
#查看定时任务
crontab -l
```

# 3、cron表达式
cron表达式，从crontab文件中抠出来的
```shell
# Example of job definition:
# .---------------- minute (0 - 59) 分钟
# |  .------------- hour (0 - 23) 小时
# |  |  .---------- day of month (1 - 31) 几号
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ... 几月
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat 星期几
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```
举几个例子吧：

```shell
# 每天晚上23点30分执行爬虫脚本
30 23 * * * /usr/local/bin/python3 /opt/data/crawl.py
# 每五分钟执行爬虫脚本
*/5 * * * *  /usr/local/bin/python3 /opt/data/crawl.py
# 每天1点-5点每小时执行一次爬虫脚本
0 1-5 * * *  /usr/local/bin/python3 /opt/data/crawl.py
```


