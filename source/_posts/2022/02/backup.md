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
## 如何将整个操作系统备份？

​	在我还没开始查找资料的时候，我的想法就是，把根目录下的各种文件原封不动的复制进备份文件夹中就行了，但是仔细想，这样又会出现一个问题，**linux 中一切皆文件**，硬件设备什么的也是会变为文件存在于系统中的，这些东西肯定是无法备份的，因此必须考虑一些排除项。比如 `/dev `这个文件夹下存储的是 linux 的设备文件，无需备份，再比如`/proc` 这个文件夹存在于内存中，里面数据与内存相关，无需备份，因此也需排除这个目录，还有就是像`/mnt`目录啊，`/swap`等等，都需要排除进去 。

​	网上找找，思路大致都是一样的，通过某种方式，将系统根目录下的文件备份，再排除掉其中一些文件夹，之后打包就行了。

我在这里就使用了用rsync进行文件复制的方法。

整个方法的大致的流程:

* 首先根据系统目前的文件挂载，测算出生成备份文件的大小,用`dd`命令创建这个文件
* 将这个文件转换为loop设备
* 对这个设备进行分区（引导区与文件目录区）
* 挂载loop设备的引导区到/mnt，用cp命令将系统引导区内容写入，卸载loop设备
* 挂载loop设备的根目录分区到/mnt，用 rsync 命令将当前系统的数据写入，卸载loop设备
* 将loop设备转为普通文件
* 得到包含引导区与设备上文件数据的一个备份文件

方法的实现我是使用了github上 [nanhantianyi/rpi-backup](https://github.com/nanhantianyi/rpi-backup) 这个库，这个库将设备挂载到了mnt分区，mnt分区上不能有其他设备挂载，我便在执行备份脚本前将该设备卸载了。为了实现自动备份以及自动邮件通知功能，我又使用了crontab来进行定时任务执行，使用python来执行邮件的发送任务。

## python邮件发送

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
        #def send_mail(subject,content)
```

里面需要传递的参数无非就是邮件的主题与正文，由于参数可能是比较复杂的文本，包含很多空格与换行，直接在参数中传递会很麻烦，比如下面这种情况

```bash
python mail.py "ready to backup!" "your raspberry pi will start backup in 10 minutes"
```

全文也没有换行，会很不舒服。因此，我们可以写好一个shell模板，在里面写好我们想输出的信息，之后将shell模板执行结果作为变量传递给python程序。

```bash
# notice.sh
min=60
echo "backup finished!"
echo "created file: $1"
echo "file size : $2"
echo "total time :$(($3/$min))min($3s)"
echo "----part of the result----"
tail -n 5 back.log

#backup.sh
...
after_text=`/bin/bash after.sh ${file} ${file_size} ${cost_time}`
python3 mail.py "backup over!" "$after_text"
...
```

这样之后我们就可以方便的展示我们想看到的信息，同时也可以通过邮件查看备份是否正常等等。

最终的邮件结果呈现如下所示

{% asset_img mail.png %}

## CRONTAB 定时任务

有关于 CRON 的介绍在之前的博文中有提到过

***

{% post_link cron cron定时任务 %}

***

因为备份系统所需要的权限比较高，所以需要使用 root 用户编辑计划任务表

```bash
# open crontab
crontab -e
# edit plan
...
00 2 *  *  0  /media/rpi-backup/back.sh  2>&1 | tee /media/rpi-backup/back.log
...

```

这一条意思就是在每周日的凌晨2点开始对系统进行备份，同时将脚本的的标准输出与错误输出同时输出到 back.log 日志文件中

需要注意的一点是，cron在执行任务的时候是不会把用户环境变量加进去的，所以很有可能会出现找不到命令的情况，因此，需要在bash脚本开头将path路径添加上去

> PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

还有需要注意的就是程序路径了，如果脚本中的所有文件操作都是绝对路径，那就不需要考虑，如果是相对路径的话，需要设定好路径或者是在脚本合适的位置做好cd切换文件目录的操作。

## 总结

这么一搞也学到了很多了，有了个新的思路。本来是想将系统的文件全都复制然后同步到某个挂载的文件夹下面的，但是看到通过使用`losetup`命令将文件作为设备挂载，这样的方法让我眼前一亮，这样就可以对一个文件进行分区，这样以来即使系统无法开机了，这里的备份文件也能够支持系统正常开机恢复，可以恢复到与之前一样。

## 参考资料

[nanhantianyi | rpi-backup](https://github.com/nanhantianyi/rpi-backup)
[菜鸟教程 | Python SMTP发送邮件](https://www.runoob.com/python/python-email.html)
[BigBubbleGum | 树莓派4B 的系统备份方法大全（全卡+压缩备份）](https://post.smzdm.com/p/apzkgne7/)