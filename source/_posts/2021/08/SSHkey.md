---
title: ssh密钥验证那些事 
date: 2021-08-28 01:16:12
tags: 
- Linux
- ssh
- github
---

***
&emsp;&emsp;第一次接触到ssh密钥还是我在捣鼓树莓派的时候，当时拿着我的树莓派PI4B在搞项目，做一些IoT之类的小系统.当时需要上传各种驱动和相关代码进行测试，单纯的命令行进行ssh连接肯定会比较麻烦，但又感觉像[putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/)、[MobaXterm](https://mobaxterm.mobatek.net/)这类的又不够美观，于是就选择了Vscode来作为IDE，它里面的remote-ssh插件可以很方便的与linux服务器进行ssh连接，只需要配置好.config文件就可以进行连接了.但是有一个问题，每次连树莓派的时候我就都得输一次密码，有时候一不小心关了终端什么的，又得重新开一遍，挺麻烦的，于是就在网上找有没有什么方法可以让vscode记录密码自动登录或者可以避开密码验证什么的，然后我就找到了这个叫做SSH key 的东西.<!-- more -->  
***
## SSH key简单介绍
&emsp;&emsp;通常我们进行SSH远程连接服务器是指定一个用户，然后再输入其密码，由服务器进行验证通过之后即可建立连接.  
&emsp;&emsp;SSH key的验证方法有一些不同，它会同时产生两个密码(密钥)，其中一个被称作公钥，另一个被称作私钥.公钥，顾名思义，就是可以公开的密钥，交由服务器保管作为加密密钥和凭证，而私钥则是自己需要秘密保管的密钥.  
&emsp;&emsp;SSH key采用了一种叫做"[非对称加密](https://baike.baidu.com/item/%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95/1208652?fr=aladdin)"的方式进行通信加密  
{% asset_img wiki.png %}  
Nicobon，[CC BY-SA 4.0](https://commons.wikimedia.org/w/index.php?curid=48484042)

其默认算法是[rsa算法](https://baike.baidu.com/item/RSA%E7%AE%97%E6%B3%95/263310?fromtitle=RSA&fromid=210678&fr=aladdin)，用户需要与服务器建立连接时，就向服务器发起请求，同时将公钥（<font color='#fdb933'>由私钥可以很容易生成公钥，但是由公钥生成私钥以目前的算力来说几乎不可能</font>）捎带上去，服务器将该密钥与留存在自己机器上的公钥相比对，如果有一致的话，就会向请求主机发送一个用该公钥加密的“challenge request”，只有私钥才能解密该“challenge request”并作出相应的回应.  
&emsp;&emsp;这一系列的过程中，只有公钥在网络上进行了传递，私钥一直保存在了本机，相比于输入密码，密码通过网络传递到服务器再进行验证的过程来说，无疑是更加安全的.
## SSH key使用
&emsp;&emsp;一般在发行版的linux中，系统在第一次初始化的时候都会生成默认的公钥和私钥对，存放路径在/etc/ssh/ 文件夹下：
{% asset_img sshkey.png sshkey %}
里面预置了许多算法，如dsa、rsa、ecdsa等，一般的sshkeys都会使用rsa算法来生成密钥对.要使用该密钥对，可以进行如下操作:  
``` 
cp /etc/ssh/ssh_host_rsa_key.pub  ~/.ssh/id_rsa
sudo chmod 600  ~/.ssh/id_rsa
# 如果有特定的端口，请用 scp -P port 指定
scp /etc/ssh/ssh_host_rsa_key   usr@host:~
# 连接到服务器
ssh usr@host
sudo cat ~/ssh_host_rsa_key  >> ~/.ssh/authorized_keys
```
***注意\*：id_rsa的权限（包括.ssh目录权限）不能过高，一般600即可，否则会报错**
{% asset_img rsaerror.png error %}

&emsp;&emsp;此外，生成ssh-key密钥对还可以采用SSH-keygen的命令，这里也更推荐使用这种方法来使用ssh密钥，该方法可以在该用户目录下生成.ssh文件夹.并在该文件夹下存放密钥，不必担心文件权限的问题.
```
ssh-keygen -t rsa -C  "your comments" 
```
-t 指代加密方式，方式有[ dsa | ecdsa | ecdsa-sk | ed25519 | ed25519-sk | **rsa** ]默认方式即为rsa   
-C 则为注释，会显式放在公钥末尾上，可以由此辨别公钥.  
输入命令之后，系统会让你输入密钥对存放路径，默认放在~/.ssh/ 路径下，确定好历经之后就要求输入密钥的密码（一般直接回车两次跳过即可） 

```
Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again:
Your identification has been saved in /root/ssh/id_rsa
Your public key has been saved in /root/ssh/id_rsa.pub
```
将公钥复制到需要登陆的服务器上的authorized_keys之后，即可使用该ssh密钥对进行ssh密钥方式登录.  
## github SSH key添加方式
&emsp;&emsp;早在去年，github就宣布了要取消[账户密码登录](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/)的验证方式，并在今年8月13号正式实施.如今使用账号密码进行push时，github会直接拒绝验证，因此必须采取添加SSH密钥或其他的方式进行登录验证.在成功生成了SSH key密钥对之后，其实需要操作的步骤就很简单了

{% asset_img addkey.png addkey %}  

只需要将自己的id_rsa.pub中的内容复制，粘贴到key框中即可  

{% asset_img addnew.png addnew %}

可采用如下方法测试:  
```
ssh -T git@github.com
Hi mrilboom! You've successfully authenticated， but GitHub does not provide shell access.
```
之后登录github即可自动使用ssh密钥登录