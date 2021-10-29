---
title: cx_Oracle操作Oracle数据库(一):配置篇
date: 2021-08-30 10:46:51
tags: 
- python
- Oracle
---

***
&emsp;&emsp;前不久刚找到了第一份实习工作,刚进公司搞不了啥东西,也就熟悉一下环境,于是导师就给了我一份解析excel数据的活干.要求是将远程服务器中的数据拉到本地,并用python将数据进行解析,然后存入Oracle数据库中.网上查了一下,看上去使用最多的就是cx_Oracle这个库了,cx_Oracle配置起来较为复杂,刚开始的时候弄了半天才弄好.<!-- more -->  
***
## cx_Oracle 介绍
***
详细信息请参阅[cx_Oracle官方文档](https://cx-oracle.readthedocs.io/en/latest/)
***
&emsp;&emsp;cx_Oracle是一个Python扩展模块,支持Python访问Oracle数据库.

### 结构
&emsp;&emsp;该拓展模块大致的结构如下图  
{% asset_img cx_Oracle_arch.png cx_Oracle_arch %}  
&emsp;&emsp;开发者通过编写python程序来调用cx_Oracle模块,而cx_Oracle在内部动态加载 Oracle 客户端库以访问 Oracle 数据库,连接的数据库可以在本地,也可以在远程服务器.  
&emsp;&emsp;图中与Oracle数据库真正进行通信连接的是最下方的Oracle client libraries，也就是oracle客户端的一些库，由此可以看出,要想通过cx_Oracle操控Oracle数据库,仅仅安装cx_Oracle这个模块是不够的，必须用到Oracle的客户端.

### instant client 
&emsp;&emsp;客户端一般使用[oracle instant client](https://www.oracle.com/database/technologies/instant-client/downloads.html),cx_Oracle在进行操作时只需要用到oracle客户端中的一些库,因此选择Basic Light进行下载即可,注意选的时候不要选错了版本，版本选择最新的就好,以Windows.x64为例，选择此项，高版本的向下一般都可以兼容.  
{% asset_img download.png  %}

下载完后解压,之后将文件夹路径添加到系统变量中即可完成oracle client的准备.

## cx_Oracle 安装  

_注意:文章中使用的版本为 **python3.6+**_  
&emsp;&emsp;由于本人python脚本的编写是在pycharm中进行的,因此直接通过GUI添加了cx_Oracle这个模块,如果是命令行,则可以用

```
pip3 install cx-oracle
```
来进行安装.如果安装失败,很有可能是网络原因,可以选择换pip镜像源或者[下载离线包](https://pypi.org/project/cx-Oracle/#files)来安装

```sh
#转移到存放下载好的模块的地方
cd .../modules/
pip3 install cx_Oracle-8.2.1-cp36-cp36m-manylinux1_x86_64.whl
```

## 使用cx_Oracle连接数据库
安装完cx_Oracle库之后即可开始使用cx_Oracle操作数据库  ，下面是一个最简易的cx_Oracle连接数据库的方式。
```python
import cx_Oracle
DB = 'username/password@ip:port/SERVICE_NAME'
conn = cx_Oracle.connect(DB)
try:
    sql = "select to_char(sysdate,'yyyy-mm-dd hh24:mi:ss') from dual"
    with conn.cursor() as cur:
        result = cur.execute(sql).fetchall()
    print(result[0][0])
    #2021-08-31 09:47:16
finally:
    conn.close()
```

如果配置无误，最后数据库会成功返回当前的日期时间.


如果环境变量中配置过数据库的TNS_ADMIN路径并配置好了tnsnames.ora文件，就可以通过如下方式连接:

```sh
#tnsnames.ora文件配置示例
mydb =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = x.x.x.x)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = orcl)
    )
  )
  
```

```python
connection = cx_Oracle.connect(user=user, password=userpwd, dsn="mydb",
                               encoding="UTF-8")
```

只需要配置用户名和密码即可进行连接。

## 附一些常见错误：  
### DPI-1047：这一类报错通常是cx_oracle一些本地的配置的问题
```sh
cx_Oracle.DatabaseError: DPI-1047: Cannot locate a 64-bit Oracle Client library: "The specified module could not be found"
```

**原因** 

* 系统路径中没有添加Oracle的客户端路径  
* 系统安装的cx_Oracle版本与Oracle客户端版本不匹配（ x32 | x64 ）

**解决方法**  

如果路径有误:  
&emsp;&emsp;添加相应的路径到环境变量中  

* Windows:  
    PATH中添加...\instantclient_11_2文件夹即可
* Linux:
```sh
#永久将该路径存入系统变量中
sudo sh -c "echo /opt/oracle/instantclient_21_1 > /etc/ld.so.conf.d/oracle-instantclient.conf"  
sudo ldconfig
```
如果是版本不匹配，那么就重新下载位数匹配的客户端客户端再次进行尝试.  

### ORA-12514：这一类报错通常是数据库服务方面的问题
```sh
cx_Oracle.DatabaseError: ORA-12514: TNS:listener does not currently know of service requested in connect descriptor
```

**原因** 

* Oracle连接服务名设置出错  

**解决方法**  

* 修改Oracle连接的服务名，服务名可以通过oracle命令行查询

  ```sql
  SQL> SHOW PARAMETER service_names
  NAME                                 TYPE        VALUE
  ------------------------------------ ----------- ------------------------------
  service_names                        string      orcl
  ```

  

  可以看到这里服务名就是orcl(一般默认服务名即为orcl).修改后应该就可以成功连接了.

