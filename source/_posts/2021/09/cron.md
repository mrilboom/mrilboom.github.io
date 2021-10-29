---
title: cron计划任务
date: 2021-09-09 10:06:03
tags:
- Linux
---
***
&emsp;&emsp;之前在做数据解析自动化导入的时候,因为前端的调度系统出现了问题,导致无法连接上远程的shell,因此只能在本地进行任务的自动导入,用到的就是 cron 这个工具.<!-- more -->
***
***本文介绍的是Linux系统中的 cron 定时任务， Java 中的 Spring 的 cron 会有所不同，不过基本上是大同小异***
## cron 介绍

在 Linux 系统中 crontab 是用来定期执行程序的命令,当安装完成操作系统之后,默认便会启动此任务调度命令. cron每分钟会定期检查是否有要执行的工作,如果有要执行的工作便会自动执行该工作.

### cron 和 crontab 的区别

cron 是 Linux 进行定期执行程序的一个工具,它提供给用户一个条 crontab 命令来进行调度使用,而 crontab 则是 cron 提供给用户进行计划任务管理的一条命令.

## 使用 cron

### cron 安装

&emsp;&emsp;一般 Linux 系统中都会默认安装并自动启用 cron,可输入 `cron`命令进行查看,如果没有安装,则可以进行手动安装:

```bash
# CentOS
yum install vixie-cron
yum install crontabs
# Ubuntu
apt-get install cron
# 启动服务
service cron start
# 设置自启动
systemctl enable cron
```

### cron 使用

&emsp;&emsp;启动了 cron 服务之后,就可以通过 crontab 命令来进行管理使用,cron通过读取 crontab 配置表来进行计划的执行,对于用户而言,配置表在`/var/spool/cron/crontabs/username`中,要修改表只需要通过执行`crontab -e`就可对里面的数据进行修改.而对于系统而言,计划任务的配置在`/etc/crontab`中,直接通过修改该文件即可进行全局配置.

&emsp;&emsp;cron 基本的配置如下:

```bash
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
t1 t2 t3 t4 t5 (user-name) command
```

其中,command 即为打算计划执行的命令,t1、t2、t3、t4、t5 分别为执行日期分钟、小时、日、月和星期的设置,以这些数据即可具体到一年中的每一天具体的某时某分的一个时间点.其中:

* `*`代表所有时间
* `-`代表时间段区间
* `,`代表单独时间
* `/`代表每隔多少时间

```bash 
00 8 *  *  1-5  wall Good morning, programmer!
```

比如上面的这条命令就可以在每周工作日早上8点钟向所有登录终端的用户发送一条消息.

如果想设置工作日,从8点起到18点,每隔2小时发送一条消息,可以改成:

```bash
00 8-18/2 *  *  1-5  wall Hello, programmer!
```

这些参数中,有两个比较特别: t3 和 t5 ,比如:

```bash
00 8 9  *  5  wall Good morning, programmer!
```

2021年9月9日是周四,这条命令会在每月的9号和每周的周五执行,具体把时间分组的话可以分为G1:t1,t2（几时几分）,G2:t3,t4（几月几日）,G3:t5（周几）,最后的执行时间就是R1:(G1,G2)与R2:(G1,G3)取并集(∪).而每个G、R之内的时间都是取交集(∩)的.

## 测试

&emsp;&emsp;在网上找到了一个可以在线进行 cron 表达式测试的网站:[crontab 执行时间计算](https://tool.lu/crontab/),如果对于时间设置不熟悉的可以在这里多尝试一下.

## 模板

这里提供了一些cron常用的模板以供参考

| 效果                              | cron模板(中括号中数字为参数顺序) | cron预览             | 默认值     |
| --------------------------------- | -------------------------------- | -------------------- | ---------- |
| 每15分钟构建一次                  | 0/{} * * * *                     | 0/15 * * * *         | 15         |
| 周1到周5，8点~17点，2小时构建一次 | 0 {2}-{3}/{4} * {0}-{1}          | 0 8-17/2 * * MON-FRI | 1,5,8,17,2 |
| 每天8~17点，2小时构建一次         | 0 {0}-{1}/{2} * *                | 0 8-17/2 * * *       | 8,17,2     |
| 每天8点构建一次                   | 0 {} * * *                       | 0 8 * * *            | 8          |

## 参考资料

* [菜鸟教程 | Linux crontab 命令](https://www.runoob.com/linux/linux-comm-crontab.html)
* [再見理想 - 博客园](https://www.cnblogs.com/intval/p/5763929.html)

