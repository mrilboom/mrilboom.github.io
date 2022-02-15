---
tags:
  - things
mathjax: true
comments: true
hidden: false
date: 2022-02-14 14:31:00
title: 使用树莓派与智能电视打造家庭影音
---

***
春节假期期间，在家总想搞点事情，翻箱倒柜找到了尘封已久的移动硬盘，于是就拿它搞了点事情。。。<!-- more -->

***
2022年春节前夕杭州地区出现了疫情，公司也很早的就发布了居家办公的通知，我便回了家，进行了2+14的居家健康观察。在家不能出去也是无聊啊，于是就只能看电视，翻了一大堆电视机上的app，搜索资源啥也搜不到，这怎么能忍？于是乎我就想实现电脑上找片——电视上观看的优雅操作。

## 使用的设备列表

* 树莓派4B（或其他可拓展与交互的linux终端）
* 移动硬盘（2T 机械）
* 智能电视（或者智能机顶盒，只要能安装第三方apk即可）

## 部署

### 树莓派

#### SMB服务

树莓派是整个系统的核心，当然。也可以用其他linux终端进行，首先将移动硬盘与树莓派相连接，执行`fdisk -l`，查看硬盘挂载信息，硬盘分区我是分了2块，一块为 linux 的 ext4 文件系统，另一块为windows的 ntfs 文件系统，每块都为500G，预留下 1T 方便以后的拓展部署。

为了方便服务器重启之后快速挂载硬盘，我编写了一个 mount.sh 的脚本来实现硬盘的挂载操作。当然，将挂载信息直接写入系统 /etc/fstab 里面也可以，但是我之前被这个给坑过，之前也是将挂载操作写进了 fstab，有一次重启之后系统却进不去了，我很疑惑，我也没修改什么系统文件啊，后面拔下 sd 卡，在电脑上读取日志之后发现原来是 mount 出了问题，如果服务器重启之后，硬盘被拔掉或者硬盘出错、接口松动，服务器就无法顺利挂载，那么就会一直卡在这一步，无法进入系统，所以我选择了先进系统再进行挂载。

挂载之后，需要配置文件共享服务

```bash
sudo apt-get install samba
```

安装完 samba 之后，打开配置文件，添加自己的服务进行配置

```bash
vi /etc/samba/smb.conf
```

```
[samba]				# 括号内的文字即为samba的服务名
path = /mnt			# 文件共享的目录，此处设置为硬盘挂载目录即可
available = yes
writeable = yes		# 可写
security = share
browseable = yes
guest ok = yes		# guest访问，无需登录
```

保存之后重启一下 samba 的服务

```bash
sudo service smbd restart
```

此时，同一局域网内的设备即可通过 smb 服务访问到 /mnt 目录下的的文件，可以用电脑测试一下连通性(电脑需要与服务器在同一个网络中)，直接打开“此电脑”，选择映射网络驱动器，输入`\\服务器ip`然后点击浏览即可看到服务器smb服务中共享的内容

{% asset_img f.png %}

此时服务器的smb服务已经成功可用，接下来该部署的就是服务器的下载功能了。

#### ARIA2服务

aria2 服务能够实现离线下载，磁力链、种子等都可以通过该服务实现下载。

首先服务器需要安装 aria2

```
apt-get install aria2
```

之后就是配置 aria2 的 aria2.conf 文件

```bash
sudo mkdir /etc/aria2  #新建文件夹
sudo touch /etc/aria2/aria2.session   #新建session文件
sudo chmod 777 /etc/aria2/aria2.session   #设置aria2.session可写
sudo vi /etc/aria2/aria.conf    #创建并编辑配置文件	
```

配置文件aria2.conf

```
dir=/mnt/windows/Movie
disable-ipv6=true
#文件空间预分配，如果无专门做种需求，则无需设置预分配，设置none即可
file-allocation=none
#打开rpc的目的是为了给web管理端用
enable-rpc=true
rpc-allow-origin-all=true
rpc-listen-all=true
rpc-listen-port=6800
#断点续传
continue=true
input-file=/etc/aria2/aria2.session
save-session=/etc/aria2/aria2.session

#最大同时下载任务数
max-concurrent-downloads=20
save-session-interval=120

# Http/FTP 相关
connect-timeout=120
#lowest-speed-limit=10K
#同服务器连接数
max-connection-per-server=10
#max-file-not-found=2
#最小文件分片大小, 下载线程数上限取决于能分出多少片, 对于小文件重要
min-split-size=10M

#单文件最大线程数, 路由建议值: 5
split=10
check-certificate=false
#http-no-cache=true


```

保存文件之后，即可启动该aria2服务。

```bash
aria2c --conf-path=/etc/aria2/aria2.conf -D
```

该命令即可将aria2服务设置在后台运行

为了能够实时监测该服务的内容，我开了一个screen以供查看输出

```bash
screen -S ARIA #创建名为ARIA的screen
aria2c --conf-path=/etc/aria2/aria2.conf
```

至此，linux服务器部分的配置就此结束了，接下来是智能电视终端部分。

### 电视终端

电视终端方面我选择的是 KODI ( [中文网](http://www.kodiplayer.cn/) | [官网](https://kodi.tv/) )应用,KODI是一个开源的播放器，功能强大，支持很多的平台

{% asset_img platforms.png %}

这边我们选择 android 平台的 apk 下载，下载完成后就可以进入系统进行安装。安装完毕之后即可进入系统。

打开KODI，之后添加smb数据源，输入服务器的 ip 即可检测到smb共享的目录，之后即可播放存贮在硬盘中的电视电影。

### PC端

PC端主要是需要可视化的进行电视电影的下载管理，我们所使用的 aria2 离线下载服务可以通过 https://aria2c.com/ 这个web控制台进行管理

{% asset_img rpc.png %}

开启aria2服务，进入这个控制台，设置好对应的 JSON-RPC 地址之后即可连接到服务器。在控制台里可以以 GUI 形式新增下载项目，这样就可以不用在命令行里进行复杂的操作了。

***JSON-RPC的地址默认为服务器的6800端口*** 

```
http(s)://服务器ip:6800/jsonrpc
```



{% asset_img aria2.png %}

## 总结

这一个流程下来基本上已经实现了通过电脑端控制，然后电视端收看的功能，虽然还是会有各种不太方便的地方，但是也勉强够用了。同时KODI这个软件也是功能很强大，支持各种插件，很多电影或者剧集下载下来可能没有字幕，KODI可以安装字幕的插件，支持在线搜索字幕，同时也支持各种电视电影的简介、评分等获取等，更多的功能就待大家自行发掘了。


