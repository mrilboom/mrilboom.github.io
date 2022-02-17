---
tags:
  - Linux
  - python
mathjax: true
comments: true
hidden: false
date: 2022-02-17 15:26:48
title: 树莓派的全备份
---
***
前两天看了发现自己树莓派日志文件逐渐变多了，系统的 journal 已经打到了4GB大小，突然想到，万一什么时候系统不小心被我玩崩了，那该咋办，于是我就想备份一下自己的树莓派<!-- more -->

***
​	在我还没开始查找资料的时候，我的想法就是，把根目录下的各种文件原封不动的复制进备份文件夹中就行了，但是仔细想，这样又会出现一个问题，**linux 中一切皆文件**，硬件设备什么的也是会变为文件存在于系统中的，这些东西肯定是无法备份的，因此必须考虑一些排除项。比如 `/dev `这个文件夹下存储的是 linux 的设备文件，无需备份，再比如`/proc` 这个文件夹存在于内存中，里面数据与内存相关，无需备份，因此也需排除这个目录。

网上找找，思路大致都是一样的，通过某种方式，将系统根目录下的文件备份，再排除掉其中一些文件夹，之后打包就行了。

我在这里就使用了用rsync进行文件复制的方法。

整个方法的大致的流程:

* 首先根据系统目前的文件挂载，测算出生成备份文件的大小,用`dd`命令创建这个文件
* 将这个文件转换为loop设备
* 对这个设备进行分区（引导区与文件目录区）
* 挂载loop设备的引导区到/mnt，用cp命令将系统引导区内容写入，卸载loop设备
* 挂载loop设备的根目录分区到/mnt，用 rsync 命令将当前系统的数据写入，卸载loop设备
* 将loop设备转为普通文件
* 就得到了包含引导区与设备上文件数据的一个备份文件

方法的实现我是使用了github上 [nanhantianyi/rpi-backup](https://github.com/nanhantianyi/rpi-backup) 这个库，这个库将设备挂载到了mnt分区，mnt分区上不能有其他设备挂载，我便在执行备份脚本前将该设备卸载了。为了实现自动备份以及自动邮件通知功能，我又使用了crontab来进行定时任务执行，使用python来执行邮件的发送任务。

### python邮件发送

python邮件部分采用的库是 smtplib 这个内置库，整个邮件发送的流程可以分为三部分：登录、写邮件和发送邮件。

登录部分需要有邮件服务器地址、用户名和密码

```
mail_host = 'smtp.qq.com'   
mail_user = 'username'
mail_pass = 'password' #smtp服务的话需要的是授权码
```

写邮件部分需要用到 email.mime 模块，该模块能够构造邮件发送的邮件体，可以发送纯文本文件，同时也支持添加附件。可以自由组合

```python
if subject == "mail with attachment":
    attachFile = 'file'
    attachApart = MIMEApplication(open(logFile,'rb').read())
    attachApart.add_header('Content-Disposition', 'attachment', filename=attachFile)
    msg = MIMEMultipart()
    msg.attach(attachApart)
    msg.attach(MIMEText("content"))
else:
    msg = MIMEText("content") #纯文本邮件
msg['Subject'] = subject
msg['From'] = me 
msg['to'] = to_list 
```

发送邮件则是需要邮件的发送人(username@example.com)、接收人与邮件的邮件体

```python
try: 
    s = smtplib.SMTP_SSL(host=mail_host) 
    s.connect(mail_host) 
    s.login(mail_user,mail_pass) 
    s.sendmail(me,to_list,msg.as_string())
    s.close()
except Exception as e: 
    print(e) 
```

需要传递的参数可以这么设置：

```python
if __name__ == "__main__": 
        send_mail(sys.argv[1],sys.argv[2])
```



<font size=4 face="幼圆" color='lightblue'>to be continue···</font>

<!-- ## 参考资料

<div style="width:70%;margin:auto">{% asset_img xxx %}</div> -->
