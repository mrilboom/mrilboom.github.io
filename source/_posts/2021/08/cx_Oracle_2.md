---
title: cx_Oracle操作Oracle数据库(二):代码篇
date: 2021-08-31 16:02:09
tags:
- python
- Oracle
---

***
&emsp;&emsp;{% post_link cx_Oracle_1 上一篇 %}讲了cx_Oracle的安装以及相关的环境配置,本篇就来讲讲cx_Oracle这个模块的具体使用方法.<!-- more -->
***
&emsp;&emsp;_本文只介绍基本的一些数据库最基本的操作方法,不涉及到连结池、存储过程等操作_

***

详细信息请参阅[cx_Oracle官方文档](https://cx-oracle.readthedocs.io/en/latest/)

***

## cx_Oracle一些对象的介绍

### 在成功连接数据库之后

在连接上数据库之后,cx_Oracle会返回一个[Connection对象](https://cx-oracle.readthedocs.io/en/latest/api_manual/connection.html#connection-object),该对象用的比较多的就是`Connection.cursor()`和`Connection.close()`了,前者是用来返回一个新的*cursor* 对象,我们进行的CRUD等大部分的操作都要用到*cursor* 对象来进行操作.后者则是关闭连接的操作,每次执行完程序后,都需要将连接关闭.

### 开始前的几个小建议

* 建立完数据库连接之后,尽量使用try——finally语句,避免因为程序出错而导致数据库连接未被正常关闭的现象的出现.	

  ```python
  conn = cx_Oracle.connect(DB)
  try:
      ...	# your codes
      ...
  finally:
      conn.close()
  ```

  这样一来,即使程序运行出错,最终也会关闭连接.

* 使用*cursor*游标时也要注意,需要手动关闭游标,不过为了方便,一般有需要到时可以使用with...as...的方法来进行调用,下面的这段代码能够确保一旦代码块完成,游标就会被回收关闭.

  ```python
  with conn.cursor() as cur:
      ...	# cursor operation
      ...
  ```

  

## 使用cx_Oracle

### 使用使用cx_Oracle进行查询

执行完对应sql语句后,游标可以通过按行进行迭代来获取所要查询的数据.

```python
with conn.cursor() as cur:
	for row in cur.execute(sql):
        print(row)
```

同时也可以通过游标对象的fetch()方法来获取数据:

```python
with conn.cursor() as cur:
	cur = cur.execute(sql)
	result = cur.fetchone()
print(type(result))
# <class 'tuple'>
print(result)
```

&emsp;&emsp;这个方法首先通过游标`cursor.execute()`执行sql语句,之后通过*cursor* 的fetch方法来获取结果,通过打印输出可以看出执行语句获取的结果是以tuple元组的方式返回的,取到数据后就可以进行其他的操作了,而cur也随着代码执行结束自动销毁了.

&emsp;&emsp;除了用`cursor.fetchone()`方法之外,还可以用`cursor.fetchall()``cursor.fetchmany(num_rows)`方法来获取对应行数的数据.

&emsp;&emsp;如果返回的数据不止一条,数据则会以列表的方式返回,列表中的每一条数据都是一个元组,每个元组表示查询到的一行数据,如

```python
[("LiMing",12,1),("LiHu",13,1),("XiaoHong",12,0),("XiaoHui",12,0)]
```

通过上述方法就能获取到我们执行数据库操作后查询到的结果了.

### 使用cx_Oracle进行其他操作（DML、DDL、DCL）

&emsp;&emsp;进行其他操作一般不需要处理返回结果集,用到的执行函数也是`execute()`,因此直接编写sql语句再调用execute()方法执行即可.

&emsp;&emsp;但是有一点需要特别注意,当`cur.execute()`执行一条 SQL 语句的同时会启动或继续一个事务.默认情况下,cx_Oracle 不会将此事务提交到数据库.方法`conn.commit()`和`conn.rollback()`方法可用于显式提交或回滚事务：

```python
cursor.execute("insert ... into ... values ...)
connection.commit()
```

数据库连接conn关闭时,任何<font color='#fdb933'>未被提交的事务</font>都会被回滚.

## 使用绑定变量(Bind Variables)

&emsp;&emsp;因为工作中需要对数据进行批量写入,我就进行了python中字符串format的用法对sql语句进行修改:

```python
sql = f"""
insert into classroom values ('{name}', {age},{sex})"""
cursor.execute(sql)
```

执行也没有问题,但是在我一次查看文档时发现了一段话  

**_Never concatenate or interpolate user data into SQL statements:_**

```python
did = 280
dnm = "Facility"

# !! Never do this !!
sql = f"""insert into departments (department_id, department_name)
          values ({did}, '{dnm}')"""
cursor.execute(sql)
```

文档中说到千万不要直接用格式化字符串的方式提交sql命令,理由很简单,因为这样会存在sql注入的问题,把整句sql的语句交给用户来定义肯定是不理性的,之后看了一下文档,解决方法并不复杂,cx_Oracle提供了一个预编译传参的手段,将sql语句的命令预先编译,之后再将参数传入,以此来避免恶意的sql注入问题,我们可以通过`cur.execute(sql,bind_params)`这条命令来代替之前不安全的方法,bind_params是一个参数列表,比如之前的插入语句就可以修改为这样:

```python
sql = """
insert into classroom values (:name, :age,:sex)
"""
bind_params = ["XiaoSe",8,1]
cur.execute(sql,bind_params)
```

&emsp;&emsp;绑定变量方式可以按照位置、名称、类型等绑定,详细内容参阅[Using Bind Variables](https://cx-oracle.readthedocs.io/en/latest/user_guide/bind.html#using-bind-variables).

## 数据批处理

cx_Oracle为提供了数据批量写入或更新的功能,使得sql语句只用编译一次即可,每次只需更改变量即可,适用于大批量的数据写入或更新的情况.下表将用于示例：

```sql
create table CLASSROOM
(
    NAME VARCHAR2(10),
    AGE  NUMBER(4),
    SEX  NUMBER(1)
)
```

&emsp;&emsp;将下面6条数据数据一次性插入到数据库中,代码如下:

```python
data = [
    ['XiaoMing',13,1],
    ['XiaoHong',12,0],
    ['XiaoHui',13,0],
    ['XiaoMei',13,0],
    ['LiBai',14,1],
    ['XiaoSe',13,1]
]
cursor.executemany("insert into CLASSROOM values (:1, :2, :3)", data)
```

&emsp;&emsp;该方法`executemany()`一次就可以将这批数据写入数据库,避免了5次往返写入,在大批量的数据上效果显著.

### 使用批处理时需要注意的一个问题

&emsp;&emsp;在我做解析excel文件时,想要进一步提高数据库写入速度,我就想用`executemany()`的批处理来优化,将读取到的每一行以`append()`的方式放入一个列表中,然后最终通过`executemany()`来一步完成,但是中间却报错了

```
TypeError: expecting number
```

&emsp;&emsp;这一看,显然是数据格式不对了,但是之前用`execute()`执行的时候从来没有出错,在仔细分析后,我找到了造成问题的原因,下图是我之前读取数据的excel原表（数据已做更改）

{% asset_img excel.png %}

&emsp;&emsp;其中中间的几行数据并没有信息,在excel读取数据的时候会把空格读取成`""`  
&emsp;&emsp;还是以之前的*CLASSROOM* 表为例,excel表如下：

{% asset_img classroom.png %}

&emsp;&emsp;python从excel中读取到的数据如下:

```python
data = [
    ['XiaoMing',13.0,1.0],
    ['XiaoHong',12.0,0.0],
    ['XiaoHui',"",0.0],
    ['XiaoMei',13.0,0.0],
    ['LiBai',14.0,1.0],
    ['XiaoSe',13.0,1.0]
]
```

&emsp;&emsp;可以看到第4行中`XiaoHui`她的年龄数据因为缺失变成了`""`,与其它的数字类型不同,因此才造成的错误.而后我又尝试将插入的数据改成:

```python
data = [
    ['XiaoMing',"",1.0],
    ['XiaoHong',12.0,0.0],
    ['XiaoHui',13.0,0.0],
    ['XiaoMei',13.0,0.0],
    ['LiBai',14.0,1.0],
    ['XiaoSe',13.0,1.0]
]
```

&emsp;&emsp;使得第一行数据缺失,而后报错内容为

```
TypeError: expecting string or bytes object
```

&emsp;&emsp;至此,我也算是知道了这个报错的出现以及之前按条插入数据时没有出现错误的原因了

&emsp;&emsp;在执行`execute()`命令时,程序会自动按照输入的数据来编译出一条sql命令以适应数据库的表,之前每一次执行都会单独编译一条sql语句,例如输入了` ['XiaoHong',12.0,0.0],`这一行数据,程序就会将对应的`string`,`float`,`float`转化成数据库中的所需求的`string`,`int`,`int`这样的格式,在下一轮迭代中,程序读取到了`string`,`string`,`float`,那么又会产生一个从`string`,`string`,`float`到`string`,`int`,`int`这样的映射以支持不同数据类型的输入.

&emsp;&emsp;然而在执行`executemany()`命令时,sql命令**只会编译一次**,所以它只能支持一组数据关系映射,它会读取列表中的第一组元素产生数据映射,所以在数据为

```python
data = [
    ['XiaoMing',"",1.0],
    ['XiaoHong',12.0,0.0],
    ['XiaoHui',13.0,0.0],
    ['XiaoMei',13.0,0.0],
    ['LiBai',14.0,1.0],
    ['XiaoSe',13.0,1.0]
]
```

的时候,它就会要求其中输入的`age`栏都是`string`类型,当遇到`'XiaoHong'`的`12.0`时,便会出错退出.

&emsp;&emsp;知道了出错的原因那就好办了,只需要在读取excel表数据时将数据强转为对应于数据库表的类型就可以了,之后就可以用`executemany()`的方法进行处理数据了.

## 使用过程中可能会遇到的一些错误

### ORA-00911: invalid character

出现这个问题很有可能是sql句末出现了分号`;`,在通过cx_Oracle对数据库进行操作时,所有的sql语句都不需要加最后的分号,去掉分号再尝试一遍即可.

### ORA-00984: column not allowed here

这个问题就是因为插入的字符串没有加单引号,看看代码是否写成了类似于

`sql = f""" insert into classroom values ({name}, {age},{sex})"""`

的样子,出现这个原因大概率是因为使用了格式化字符串的方法来编写sql语句,所以这里还是再强调一遍推荐使用 **[绑定变量](#%E4%BD%BF%E7%94%A8%E7%BB%91%E5%AE%9A%E5%8F%98%E9%87%8F-Bind-Variables)** 的方法进行sql语句编写

### 程序执行完后数据并未写入数据库

目前没有遇到过如此问题,不过记得关闭连接之前先提交事务`commit()`,该问题很有可能是因为事务忘提交而导致的事务回滚造成的.

