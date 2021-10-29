---
tags:
  - Oracle
  - PL/SQL
comments: true
hidden: false
date: 2021-09-24 17:29:24
title: (续)Pl/SQL 学习笔记Oracle 11g 从入门到精通第4章)
---
***
&emsp;&emsp;之前讲了大体上的plsql语法结构，这里则继续对plsql的存储过程、函数等内容进行具体的介绍，这一部分由于本人与不是很理解，因此在很多地方将书中的内容直接搬了过来，以便日后查阅学习<!-- more -->

***
## 过程

之前介绍的代码结构什么的，所创建的 PL/SQL 程序都是匿名的，其缺点是每次执行时都要被重新编译，没有存在数据库中，不能被其他 PL/SQL 块调用. Oracle 允许在数据库的内部创建并存储编译过的 PL/SQL 程序，以便随时取用调用. 

### 创建过程

过程是用来完成一系列操作的语句，创建方式如下：

```plsql
CREATE [OR REPLACE]  PROCEDURE<过程名>(
	<参数 1>,[方式 1]<数据类型 1>，
	<参数 2>,[方式 2]<数据类型 2>
    ...
)
IS | AS
PL/SQL 过程体;
```

如下是一段动态观察TEACHERS表中不同性别的人数的存储过程:

```plsql
SET SERVEROUTPUT ON FORMAT WRAPPED 
--设置格式WRAPPED 这个会默认以set linesize 为分割切断以后在新的一行显示
CREATE OR REPLACE PROCEDURE count_num
	(in_sex in TEACHERS.SEX%TYPE) --入参
AS	out_num NUMBER;
BEGIN
	IF in_sex='M' THEN
		SELECT count(SEX) INTO out_num
		FROM TEACHERS
		WHERE SEX='M';
		dbms_output.put_line('NUMBER of Male Teachers:'||out_num);
		ELSE
			SELECT count(SEX) INTO out_num
			FROM TEACHERS
			WHERE SEX='F';
			dbms_output.put_line('NUMBER of Female Teachers:'||out_num);
		END IF;
	END count_num;
```

可以直接在 PL/SQL 命令窗口输入，同时也可以以文本格式存储好，然后在命令窗口使用 @ 或者 START 来载入存储过程. 

过程成功创建之后，控制台就会输出`Procedure created`,此时我们定义的过程就成功地载入了数据库内存中，可以随时调用. 

### 调用过程

调用过程的命令是 EXECUTE ,我们可以通过

```plsql
EXECUTE <过程名>(<参数>);
```

来进行调用，例如上面的 `count_num`过 程，我们进行调用:

```plsql
SQL> EXECUTE count_num('F');

NUMBER of Female Teachers:100

PL/SQL procedure successfully completed

SQL> EXECUTE count_num('M');

NUMBER of Male Teachers:100

PL/SQL procedure successfully completed
```

过程成功的输出了男女教师的数量. 

### 删除过程

语法：

```plsql
DROP PROCEDURE <过程名>;
```

以删除上面创建的`count_num`过程为例：

```plsql
SQL> DROP PROCEDURE count_num;

Procedure dropped
```

### 过程的参数类型及传递

#### in、out、in out参数类型

这些输入(输出)类型的参数，程序执行时会要求(或输出)该参数. 

```plsql
CREATE OR REPLACE PROCEDURE double  --将一个整数加倍
(
       in_num in NUMBER,     --入参
       out_num out NUMBER,      --出参
       in_out_num in out NUMBER  --出入参公用同一变量
  ) 
AS
BEGIN
  out_num:=in_num*2;
  in_out_num:=in_out_num*2;
  END double;
```

## 函数

函数一般用于计算或者返回一个值，函数的调用是表达式的一部分，过程的调用是一条 PL/SQL 语句. 

函数与过程比较大的区别就是函数调用时需要用表达式，过程只需要过程名，函数必须有返回值，过程则没有规定. 

### 创建函数

创建函数语法如下:

```plsql
CREATE [OR REPLACE] FUNCTION<>(
	(<参数 1>,[方式 1]<数据类型 1>，
     <参数 2>,[方式 2]<数据类型 2>
    ...)
RETURN <表达式>
IS|AS
PL/SQL 程序体	--其中必须要有RETURN 子句
```

在其中，RETURN 在声明部分需要定义一个返回参数的类型，而在函数体中必须有一个RETURN 语句，<表达式>就是要函数返回的值. 当语句执行时，如果表达式的类型与定义不符，该表达式将被替换为函数定义子句 RETURN 中指定的类型. 同时，控制将立即返回到调用环境.但是，函数中可以有一个以上的返回语句. 如果函数结束时还没有遇到返回语句，就会发生错误.

通常函数只有 in 类型的参数.

使用函数的方式来统计男女教师数量:

```plsql
CREATE OR REPLACE FUNCTION count_num
(in_sex in TEACHERS.SEX%TYPE)
	return NUMBER
AS
	out_num NUMBER;
BEGIN
	IF in_sex='M' THEN
		SELECT count(SEX) INTO out_num
	FROM TEACHERS
	WHERE SEX = 'M';
	ELSE
		SELECT count(SEX) INTO out_num
	FROM TEACHERS
	WHERE SEX = 'F';
	END IF;
	RETURN(out_num);
END count_num;
```

函数的创建方式与过程的创建方式完全一致，创建成功后Oracle同样会返回`Function created`.

### 函数调用

函数可以在程序块中调用：

```plsql
DECLARE
	m_num NUMBER;
	f_num NUMBER;
BEGIN
	m_num:=count_num('M');
	f_num:=count_num('F');
END;
```

同时也可以通过使用

```sql
SELECT count_num('M') male from dual;
SELECT count_num('F') female from dual;
```

直接调用.

### 删除函数

直接使用命令

```plsql
DROP FUNCTION count_num;
```

来进行删除即可.

## 程序包

程序包简称包，用于将逻辑相关的 PL/SQL 块或元素(变量、常量、自定义数据类型、异常、过程、函数、游标)等组织在一起，作为一个完整的单元存储在数据库中，用名称来表示程序包. 它具有面向对象的程序设计语言的特点，是对 PL/SQL 块或元素的封装. 程序包类似于面向对象中的类，其中变量相当于类的成员变量，而过程和函数就相当于类中的方法. 

### 基本原理

程序包有两个独立的部分包：说明部分和包体部分. 这两部分独立的存储在数据字典中. 说明部分是包与应用程序之间的接口，只是过程、函数、游标等的名称或首部. 包体部分才是这些过程、函数、游标的具体实现. 包体部分在开始构建应用程序框架时可暂不需要. 一般而言，可以先独立地进行过程和函数的编写，当其较为完善之后，在逐步地将其按照逻辑相关性进行打包. 

在编写程序包时，应将公用的、通用的过程和函数编写进去，以便再次共享使用，Oracle 也提供了许多程序包可供使用. 为了减少重新编译调用报的应用程序的可能性，应该尽可能地减少包说明部分的内容，因为对包体的更新不会导致重新编译包的应用程序，而对说明部分的更新则需要重新编译每一个调用包的应用程序. 

### 包说明部分

说明部分创建格式如下：

```plsql
CREATE PACKAGE<包名>
IS
变量、常量及数据类型定义;
游标定义头部;
函数、过程的定义和参数的列表及返回类型;
END<包名>;
```

示范:

```plsql
CREATE PACKAGE my_package
IS
	man_num NUMBER;
	woman_num NUMBER;
	FUNCTION F_count_num(in_sex in TEACHERS.SEX%TYPE)
	RETURN NUMBER;
	PROCEDURE P_count_num
	(in_sex in TEACHERS.SEX%TYPE,out_num out NUMBER);
END my_package;
```

### 包体部分

包体部分是包说明部分中内容的的具体定义，格式如下：

```plsql
CREATE PACKAGE BODY<包名>
AS
游标、函数、过程的具体定义；
END<包名>;
```

以上述包说明部分为例，包体应为：

```plsql
CREATE PACKAGE BODY my_package
	AS
	FUNCTION F_count_num	--函数定义
	(in_Sex in TEACHERS.SEX%TYPE)
	RETURN NUMBER
	AS
    	out_num NUMBER;
	BEGIN
		IF in_sex='m'	THEN
			SELECT count(SEX) INTO out_num
			FROM TEACHERS
			WHERE SEX='m';
		ELSE
			SELECT count(SEX) INTO out_num
			FROM TEACHERS
			WHERE SEX='f';
		END IF;
		RETURN(out_num);
	END F_count_num;
	PROCEDURE P_count_num	--过程定义
	(in_sex in TEACHERS.SEX%TYPE,out_num out NUMBER)
AS
BEGIN
	IF in_sex='m' THEN
		SELECT count(SEX) INTO out_num
		FROM TEACHERS
		WHERE SEX='m';
		
		ELSE
			SELECT count(SEX) INTO out_num
			FROM TEACHERS
			WHERE SEX='f';
		END IF;
	END P_count_num;
END my_package;
```

### 调用包

调用包的方式为:

* 包名.变量名(常量名)
* 包名.游标名
* 包名.函数名(过程名)

一旦包创建之后，便可以随时调用其中的内容. 

### 删除包

使用`DROP PACKAGE my_package` 即可.

<font size=4 face="幼圆" color='lightblue'>to be continue···</font>

## 参考资料

钱慎一  张素智 | Oracle 11g 从入门到精通

[jolingcome | plsql 程序包的创建与应用](https://blog.csdn.net/u012474716/article/details/79788387)

