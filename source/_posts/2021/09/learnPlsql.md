---
tags:
  - Oracle
  - PL/SQL
comments: true
hidden: false
date: 2021-09-22 15:03:39
title: Pl/SQL 学习笔记(Oracle 11g 从入门到精通第4章)
---
***
&emsp;&emsp;普通的sql语句只能进行简单的查询和一些设置, 单靠sql很难实现一些复杂的逻辑, 正是因为原来的sql语句局限性太高, 不能实现一些自动化的处理过程, 于是就出现了了 PL/SQL(Procedure Language & Structured Query Language, 过程化 SQL 语言), 它在原先的一些普通sql语句的基础上加入了流程控制语句, 如while、if 等循环或者逻辑语句, 这使得sql能够进行一些复杂的流程处理, 能够写出一些自动化脚本以供使用. <!-- more -->

***

***在学习过程中如果遇到 dbms_output.put_line() 没有输出的情况, 有可能是控制台输出没有打开, 试试在命令窗口设置`set serveroutput on`以开启输出显示.***

## PL/SQL 数据类型

*PL/SQL 数据类型与 ORACLE 数据类型并不是同一概念, 但是在创建表的时候使用的变量为数据库变量*

### 数字类型

* NUMBER(存储整数或者浮点数)
* PLS_INTEGER(只存储整数)
* BINARY_INTEGER(只存储整数)

这些都是基本类型, 还有很多数据库的子类型可供使用, 如 DEC、DOUBLE、INT 等, 其都是 NUMBER 的子类型, 在NUMBER之上添加了一些限制以增强可读性和操作性. 

### 字符类型

* VARCHAR2(储存变长字符串)
* CHAR(储存定长字符串)
* LONG(可变字符串, 可储存最高 2GB 的字符长度)
* NCHAR(PL/SQL 8.0+, 存储 Unicode)
* NVARCHAR2(PL/SQL 8.0+, 存储 Unicode)

### 日期类型

* DATE(世纪、年、月、天、小时、分钟、秒, 一共 7 字节)

### 布尔类型

* BOOLEAN(TRUE、FALSE、NULL)

### type 定义的数据类型

* `type<数据类型名>is<数据类型>;`

  使用 type 定义 teache_record 记录变量：

  > type teacher_record is RECORD
  > (
  > 	TID NUMBER(5)NOT NULL:=0,
  >     NAME VARCHAR2(50),
  >     TITLE VARCHAR2(50),
  >     SEX CHAR(1)
  > );
  

这就是一个自定义的记录变量, 要使用时通过`teacher teacher_record;`创建变量即可. 如果需要使用其中的变量, 通过类似于`teacher.SEX`的方式来获取即可

### PL/SQL 变量和常量

* 常量的定义方式为`<常量名>constant<数据类型>:=<值>;` 

  > PI constant NUMBER:=3.1415927;

* 变量的定义方式为<变量名><数据类型>[(宽度):=<初始值>];中括号中两个变量可以灵活选择使用：

  > LEN NUMBER (3,2);
  >
  > LEN NUMBER := 2.14;
  >
  > LEN NUMBER (3,2):=2.14;
  >
  > LEN NUMBER;

## PL/SQL 结构

* [声明部分](#声明部分-DECLARE)(DECLARE)
* [执行部分](#执行部分-BEGIN…END)(BEGIN...END)
* [异常处理部分](#异常处理部分-EXCEPTION)(EXCEPTION)

总体结构如下：

```plsql
DECLARE(Optional) ...
BEGIN(Required) ...
EXCEPTION(Optional) ...
END;
```



### 声明部分(DECLARE)

PL/SQL中的声明部分是可选的, 一般用来进行定义类型和变量、声明变量、声明函数、游标等操作. 

比较基本的用法: 

```plsql
DECLARE
     i NUMBER(4) := 157;
     j NUMBER(6);
     c VARCHAR2(200) := 'HELLO,WORLD'; 
     d DATE := sysdate; 
     b BOOLEAN := TRUE;
BEGIN
...
END;
```



在声明的时候就可以直接对变量进行赋值, 在plsql中, 赋值操作为`:=`, 如果声明时没有进行赋值, 则赋值操作只能在执行部分进行处理. 

进一步的操作, 在需要循环取数据库中记录时可能会用到：	

```plsql
DECLARE
	i emp%rowtype;
	j emp.ename%type;
	k j%type;
BEGIN
...
END;
```

上面的例子中, i 被定义为了emp表中的数据行的数据类型, 一个 i 的值就是一条记录, j 被定义为 emp 表中的 ename 的类型, 即 j 的类型与 emp 中 ename 栏相同, 而 k 的类型同理, 与 j 的类型相同. 

### 执行部分(BEGIN...END)

这一块是每一个plsql过程中必须要有的部分, 在这里可以进行最重要的流程控制操作. 

#### 选择结构

##### IF 语句

```plsql
IF{expression 1}Then
	{do something;}
[ELSIF(expression 2)THEN
	{do something;}]
[ELSE
	{do something;}]
END IF;
```

**需要注意其中 ELSIF 并不是 ELSEIF , 也不是 ELSE IF**

* IF ... THEN 语句

```plsql
IF LEN = 2.14 THEN
	INSERT INTO teachers VALUES(NAME,LEN,AGE);
	END IF;
```

* IF ... THEN ... ELSE 语句

```PLSQL
IF ROLE = 0 THEN
	INSERT INTO teachers VALUES(NAME,SEX,AGE);
ELSE
	INSERT INTO students VALUES(NAME,SEX,AGE);
END IF;
```

* IF ... THEN ... ELSIF 语句

```plsql
IF ROLE = 0 THEN
	INSERT INTO teachers VALUES(NAME,SEX,AGE);
ELSIF ROLE = 1 THEN
	INSERT INTO students VALUES(NAME,SEX,AGE);
END IF;
```

基本上与其他编程语言没有区别, 不过需要注意的就是在 PL/SQL 中的条件判断符号用的是`=`号, 而赋值语句用的是`:=`号, 这个与大部分常用语言不同. 

##### CASE 语句

CASE语句有很多种不同的用法

有的和其他语言中的SWITCH语句类似, 在其格式如下：

```plsql
CASE A
WHEN {expression 1} THEN {seq1}
WHEN {expression 2} THEN {seq2}
...
WHEN {expression n} THEN {seqn}
[ELSE {seq}]
END;
```

如果表达式的值与下面的任何一个表达式都不匹配时, PL/SQL 就会抛出CASE_NOT_FOUND异常. 当 CASE 匹配多条表达式时, 程序会取用最后一条符合条件的表达式执行. 

示范代码(按成绩分等级)：

```plsql
...
st_grade:=CASE
			WHEN st_score<60 THEN 'C'
			WHEN st_score>80 THEN 'A'
			ELSE 'B'
		  END;
```

示范代码(按等级评论)：

```plsql
...
st_comment:=CASE st_grade
			WHEN 'A' THEN '非常优秀'
			WHEN 'B' THEN '还不错'
			ELSE '仍需努力'
		  END;
```

#### NULL 结构

假设有下列一段代码：

```plsql
DECLARE
	V_NUMBER1	NUMBER;
	V_NUMBER2	NUMBER;
	V_Result	VARCHAR2(7);
BEGIN
	IF V_NUMBER1>V_NUMBER2 THEN
		V_Result:='NO';
	ELSE
		V_Result:='YES';
	END IF;
END;
```

可以看到这里对空值进行了 IF 的条件比较, 这时候就会造成我们理解逻辑上的错误, 不过数据库 并不会报错, 数据库在执行 NULL 值的比较时, 会返回NULL,  直接跳到 ELSE 之中, 因此, 需要有 NULL 结构来判断变量是否为空. 

```plsql
DECLARE
	V_NUMBER1	NUMBER;
	V_NUMBER2	NUMBER;
	V_Result	VARCHAR2(7);
BEGIN
	IF V_NUMBER1 IS NULL oR V_NUMBER2 IS NULL THEN
		V_Result:='Unknown';
	IF V_NUMBER1>V_NUMBER2 THEN
		V_Result:='NO';
	ELSE
		V_Result:='YES';
	END IF;
END;
```

#### 循环结构

##### LOOP 语句

* LOOP ... EXIT ... END 语句

```plsql
LOOP			--开始循环
...
[IF ... THEN]	--一般 EXIT 前有条件判断语句以控制跳出条件	
EXIT			--跳出循环
...
END LOOP;		--循环结束, 继续跳到之前 LOOP 处执行
```

* LOOP ... EXIT WHEN ... END 语句

```plsql
LOOP
...
EXIT WHEN ...	--条件表达式为真时跳出循环
...
END LOOP;
```

* WHILE ... LOOP ... END

```plsql
WHILE ...		 --表达式为真时才能进入循环开始执行
LOOP
...
END LOOP;		 --跳至WHILE继续执行
```

* FOR ... IN ... LOOP ... END

```PLSQL
FOR ... IN ...	 --例FOR i IN 0..5, 即i从0到5循环, 循环执行5次后退出
LOOP
...
END LOOP;
```

##### GOTO 语句

* GOTO label;

程序执行到 GOTO 语句时, 控制程序会立即跳转标签标识的语句. label在PL/SQL中标识为`<<mark>>`,这就定义了一个名为 mark 的标签. 

和其他所有语言一样, GOTO语句是不建议被使用的, 该语句的功能完全可以由其它的条件语句或循环语句进行改写实现, 如果该语句使用不当很容易出现大问题. 



### 异常处理部分(EXCEPTION)

异常可分为系统自导异常和用户自定义异常, 这里因为书中内容较少并且自己不熟悉所以不作细讲, 用户可以主动通过关键字 Raise 来抛出异常, 或者通过函数`Raise_Application_Error `来抛出. 

## PL/SQL 游标

PL/SQL 变量一般是标量, 一组变量一次只能存放一条记录, 但是 Oracle 一次查询获取到的数据一般是一个集合, 即多条数据, 仅仅使用这样的变量对Oracle 查询到的数据进行操作并不现实, 因此 PL/SQL 就引入了游标(cursor)的概念,这里的游标就类似于 C 里面的指针, 该游标指向一个 Oracle 内存中的缓冲区, 缓冲区里面包含了要进行操作的语句. 

### 显式游标

{% asset_img process.png %}

游标的声明需要在声明部分进行, 其他 3 个步骤都在执行或者异常处理步骤中进行. 

#### 游标声明

* `CURSOR <游标名> IS SELECT <语句>;`

```plsql
DECLARE
teacher_id NUMBER(5);
teacher_name VARCHAR2(50);
teacher_title VARCHAR2(50);
teacher_sex char(1);
CURSOR teacher_cur IS
	SELECT TID,TNAME,TITLE,SEX
	FROM TEACHERS
	WHERE TID<117;
```

这样就定义出了一个名为`teacher_cur`的游标. 

#### 游标打开

* `OPEN <游标名>;`

打开游标执行的操作其实就是数据库执行定义好的 SQL 语句, 装载进缓冲区, 游标指向结果首部. 

```plsql
OPEN teacher_cur;
```

*重复打开一个游标并不会报错, 因为系统会隐式的先 close 游标, 再重新打开这个游标*

#### 游标提取

* `FETCH <游标名> INTO <变量列表>;`

* `FETCH <游标名> INTO PL/SQL 记录;`

先看一段示例：

```plsql
OPEN teacher_cur;
FETCH teacher_cur INTO teacher_id,teacher_name,teacher_title,teacher_sex;
-- 将游标指向的那一行数据放入变量中, 游标后移(只能逐个向后移, 不能向前或者跳跃移动)
```

```plsql
DECLARE 
	teacher_record TEACHERS%ROWTYPE;
BEGIN
	OPEN teacher_cur;
	FETCH teacher_cur INTO teacher_record;
	-- 将游标指向的那一行记录放入变量中
```

这两段代码实现的功能其实相同, 都将游标中的数据给取出来, 先前定义使用`ROWTYPE`进行操作时, 会比较方便, 不用依次定义表的每个字段变量, 直接按记录读取. 进行 FETCH 时, 需要注意数据的兼容性, INTO 字句中的变量类型都必须与查询的选择列表的类型相兼容, 否则将拒绝执行. 

#### 游标关闭

当所有活动集都被检索完毕之后, 游标就理应关闭, 关闭的语法为

* `CLOSE <游标名>;`

```plsql
CLOSE teacher_cur;
```

如果游标已经关闭, 但是仍然执行 FETCH 语句则会报错, 同样, 关闭一个已经关闭的游标同样也会报错. 

### 隐式游标

隐式游标类似于 Java 中的匿名内部类, 无需进行定义、打开或关闭操作

```plsql
BEGIN
	SELECT TID,TNAME,TITLE,SEX INTO teacher_id,teacher,name,teacher_title,teacher_sex
	FROM TEACHERS
	WHERE TID=113;
END;
```

这里的 select 语句就产生了一个隐式游标

以下的三类DML语句都会产生隐式游标：

```plsql
BEGIN
	INSERT INTO teachers VALUES(221,'Baoguo','Kongfu','M')
	--INSERT
	UPDATE teachers set TID=250 WHERE TNAME='Baoguo'
	--UPDATE
	DELETE FROM teachers WHERE TID=250;
	--DELETE
END;
```



### 游标属性

无论显式游标还是隐式游标, 都有如下四种属性：

* [%ISOPEN](#ISOPEN)
* [%FOUND](#FOUND)
* [%NOTFOUND](#NOTFOUND)
* [%ROWCOUNT](#ROWCOUNT)

#### ISOPEN

判断游标是否打开, 实际应用中, 在使用一个游标前, 都应当检查它的%ISOPEN属性, 查看其是否已打开, 若没有, 要打开游标再向下操做. 这也是防止运行过程中出错的必备一步. 

```plsql
IF teacher_cur%ISOPEN THEN
	FETCH teacher_cur INTO teacher_id,teacher_name,teacher_title,teacher_sex;
ELSE
	OPEN teacher_cur;
END IF;
```

隐式游标中, 引用方式为 SQL%ISOPEN , 因为隐式游标不需要打开和关闭, 它的属性总为 TRUE , 不需要进行该操作. 

#### FOUND

判断当前游标指向是否为有效行, 即游标指向的位置有无数据, 如果游标指向的地方没有数据(NULL), 就返回FALSE, 这一属性对于循环提取数据很有效. 

```plsql
OPEN teacher_cur;
LOOP
FETCH teacher_cur INTO teacher_id,teacher_name,teacher_title,teacher_sex;

	EXIT WHEN NOT teacher_cur%FOUND;
END LOOP;
```

原书中似乎编写出现了一些错误, 按照书中执行的话, 程序会进入死循环,本人在代码上进行了一些调整

{% asset_img error.png %}

#### NOTFOUND

该属性值与%FOUND属性值相反, 其他操作与FOUND完全一致. 

#### ROWCOUNT

该属性记录了游标抽取过的记录行数, 也可以理解为此时游标在缓冲区结果集的行号, 这个属性在循环中也同样很有效使得不必抽取所有记录行就可以中断游标操作. 

比如要取出规定的一部分数据：

```plsql
LOOP
FETCH teacher_cur INTO teacher_id,teacher_name,teacher_title,teacher_sex;
EXIT WHEN NOT teacher_cur%ROWCOUNT=10;
END LOOP;
```

而隐式游标执行方法如下：

```plsql
BEGIN
	INSERT INTO teachers VALUES(221,'Baoguo Ma','Kongfu','M');
	dbms_output.put_line('INSERT ROWCOUNT:'||SQL%ROWCOUNT);
	--INSERT
	UPDATE teachers set TID=250 WHERE TNAME='Baoguo Ma';
	dbms_output.put_line('UPDATE ROWCOUNT:'||SQL%ROWCOUNT);
	--UPDATE
	DELETE FROM teachers WHERE TID=123;
	dbms_output.put_line('DELETE ROWCOUNT:'||SQL%ROWCOUNT);
	--DELETE
END;
```

#### 参数化游标

游标可以传参, 在定义的时候声明即可：

```plsql
-- ACCEPT my_tid prompt 'pls input the tid'
DECLARE
	...
	CURSOR teacher_cur(PARAM NUMBER) IS
		SELECT TNAME,TITLE,SEX FROM TEACHERS 
		WHERE TID=PARAM;	--使用参数
BEGIN
	OPEN teacher_cur(my_tid);
    LOOP
    	FETCH teacher_cur INTO teacher_name,teacher_title,teacher_sex;
    	EXIT WHEN teacher%NOTFOUND;
    END LOOP;
    CLOSE teacher_cur;
END;
```

### 游标变量

正如其名, 游标变量不局限于一条sql语句, 它可以依次对应上多条 sql 查询语句

#### 声明游标变量

PL/SQL 中引用类型通过如下方式进行声明

* `REF type`

其中 type 即将要声明的引用类型, 且应是一个已经存在的类型, 因此, 游标的引用可以按照如下方式：

```plsql
TYPE <类型名> REF CURSOR
RETURN <返回类型>;
```

类型名是引用类型的名字, 返回类型则是一个记录类型

示范代码：

```plsql
DECLARE 
TYPE Ref1 IS REF CURSOR
RETURN STUDENTS%ROWTYPE;
-- 使用%ROWTYPE 获取记录类型

TYPE Rec IS RECORD(
	sname STUDENTS.sname%TYPE,
	sex STUDENTS.sex%TYPE);
TYPE Ref2 IS REF CURSOR
RETURN Rec;
-- 自定义记录类型

TYPE Ref3 IS REF CURSOR
RETURN Rec%TYPE;
-- 另一类型定义

my_ref Ref1;
-- 如此使用REF
```

上述的游标变量全是受限的，只能返回某种特定的类型，但在 PL/SQL 中，可以不声明返回类型，省略掉 RETURN语句，这样就可以被任何查询语句打开。

#### 打开游标变量

打开游标变量需要将变量与 SELECT 语句关联，语法为：

* `OPEN<游标变量>FOR<SELECT 语句>;`

当游标变量受限且返回类型不匹配时，Oracle 就会报错。

打开游标变量示例：

```plsql
DECLARE
	TYPE my_Ref IS REF CURSOR
	RETURN STUDENTS%ROWTYPE;
	myref_cur my_ref;
BEGIN
	OPEN myref_cur FOR
		SELECT * FROM STUDENTS;
END;
```



#### 关闭游标变量

操作与显式游标关闭相同

* `CLOSE <游标变量名>;`

## 参考资料

[where are you ぃ | pl/sql编程](https://blog.csdn.net/qq_34741755/article/details/81279698)

钱慎一  张素智 | Oracle 11g 从入门到精通

## 附录

### TEACHER 表 DDL

```sql
create table TEACHERS
(
  tid   NUMBER(5),
  tname VARCHAR2(50),
  title VARCHAR2(50),
  sex   CHAR(1)
)
```



### TEACHER 表数据

采用PLSQL数据生成器生成,生成器设置：

| 名称  |   类型   | 大小 | 数据                                                         |
| :---: | :------: | :--: | :----------------------------------------------------------- |
|  TID  |  NUMBER  |  5   | Sequence(1)                                                  |
| TNAME | VARCHAR2 |  50  | Firstname+' '+Lastname                                       |
| TITLE | VARCHAR2 |  50  | List('Chinese','Engilsh','Math','Geography ','Chemistry','Music','Biology') |
|  SEX  |   CHAR   |  1   | List('M','F')                                                |

这里设置提供生成好的200条数据：

<details>
<summary>点击展开sql</summary>

```sql
insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (1, 'Quentin McKean', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (2, 'Ted Costner', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (3, 'Andrea Bracco', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (4, 'Rosanne Travers', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (5, 'Alex Astin', 'Chemistry', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (6, 'Jonatha Wells', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (7, 'Sean Osmond', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (8, 'Cole Rawls', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (9, 'Karon Penn', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (10, 'Kirsten Pleasure', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (11, 'Arturo Holbrook', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (12, 'Grant Manning', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (13, 'Aaron Redford', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (14, 'Murray Kinney', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (15, 'Chris DiFranco', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (16, 'Elias Hutton', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (17, 'Leo Costner', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (18, 'Colleen McAnally', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (19, 'Alfred Price', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (20, 'Armand O''Hara', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (21, 'Max Busey', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (22, 'Celia Loggia', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (23, 'Aaron Williams', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (24, 'Ewan Pleasure', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (25, 'Fairuza Nugent', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (26, 'Mike Zappacosta', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (27, 'Christine Mitra', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (28, 'Daniel Cohn', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (29, 'Ice Trejo', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (30, 'Bo Pollak', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (31, 'Jon Polley', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (32, 'Ricky Ojeda', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (33, 'Ricky Thomas', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (34, 'Chloe Idol', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (35, 'Trace Pigott-Smith', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (36, 'Bonnie Schiff', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (37, 'Julianne Jonze', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (38, 'Davey Streep', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (39, 'Sheryl Gatlin', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (40, 'Ellen Chao', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (41, 'Saul de Lancie', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (42, 'Gran Murphy', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (43, 'Xander Miller', 'Chemistry', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (44, 'Todd Blackwell', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (45, 'Ricardo Malone', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (46, 'Molly Cazale', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (47, 'Neve Prinze', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (48, 'Cloris Rucker', 'Chemistry', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (49, 'Udo Kelly', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (50, 'Holly McDowall', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (51, 'Colin Woodward', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (52, 'Tara Dalton', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (53, 'Susan Malkovich', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (54, 'Bernie Sweeney', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (55, 'Minnie Ruiz', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (56, 'Temuera White', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (57, 'Emma Whitman', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (58, 'Rose Masur', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (59, 'Lila Lachey', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (60, 'Chad Downie', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (61, 'Boyd Fisher', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (62, 'Joey Rivers', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (63, 'Nanci Hutton', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (64, 'Rueben Snow', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (65, 'Phil McCready', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (66, 'Nicholas Busey', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (67, 'Humberto Sartain', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (68, 'Oliver Wong', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (69, 'Henry Cole', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (70, 'Gladys Peterson', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (71, 'Seann McNeice', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (72, 'Diamond Gilley', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (73, 'Val Penn', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (74, 'Juliette Almond', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (75, 'Charlize McLean', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (76, 'Mykelti Webb', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (77, 'Jonny Lee Atkins', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (78, 'Ned Starr', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (79, 'Kathy Diesel', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (80, 'Illeana Chaykin', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (81, 'Connie Costello', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (82, 'Chazz Gary', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (83, 'Joy McPherson', 'Chemistry', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (84, 'Alannah Koyana', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (85, 'Radney Epps', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (86, 'Temuera Place', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (87, 'Chad Franks', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (88, 'Larenz Pitney', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (89, 'Jonatha Keen', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (90, 'Ashley Hiatt', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (91, 'Earl Lloyd', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (92, 'Michael Nugent', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (93, 'Gwyneth Sobieski', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (94, 'Solomon Wahlberg', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (95, 'Julianna Schwimmer', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (96, 'Roger Dempsey', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (97, 'Kate Jolie', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (98, 'Ashton Shepherd', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (99, 'Liev Cage', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (100, 'Juliette Frakes', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (101, 'Bonnie Chilton', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (102, 'Sydney MacDowell', 'Chemistry', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (103, 'Josh Pollak', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (104, 'Giovanni Pollack', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (105, 'Neneh McFerrin', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (106, 'Joan Kapanka', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (107, 'Minnie Calle', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (108, 'Hookah Bradford', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (109, 'Jesse Chan', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (110, 'Praga Rizzo', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (111, 'Debra Balk', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (112, 'David Tennison', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (113, 'Clay Shepherd', 'Chemistry', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (114, 'Willie Humphrey', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (115, 'April Flatts', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (116, 'Chant? Gertner', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (117, 'Michael Ellis', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (118, 'Morris Buscemi', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (119, 'Barry Serbedzija', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (120, 'Jessica Favreau', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (121, 'Peter Napolitano', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (122, 'Sissy Cara', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (123, 'Hope Keeslar', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (124, 'Katie Del Toro', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (125, 'Maura Lonsdale', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (126, 'Gin Gore', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (127, 'Kyra Brando', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (128, 'Alfred Copeland', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (129, 'Trick Zellweger', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (130, 'Quentin Cummings', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (131, 'Lois Haysbert', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (132, 'Cole Ontiveros', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (133, 'Aidan Hersh', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (134, 'Stephen Lucas', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (135, 'Harold Torn', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (136, 'Sigourney Miller', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (137, 'Whoopi Hirsch', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (138, 'Queen McKennitt', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (139, 'Nickel Torn', 'Chemistry', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (140, 'Russell Gagnon', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (141, 'Bridgette Theron', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (142, 'Stevie Whitmore', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (143, 'Kazem Grant', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (144, 'Stephen Hawthorne', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (145, 'Rita Tempest', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (146, 'Jean-Luc Kweller', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (147, 'Anthony Snider', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (148, 'Bette Nielsen', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (149, 'Udo Lonsdale', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (150, 'Hope Davidson', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (151, 'Bradley Lineback', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (152, 'Tanya Brody', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (153, 'Alice Wills', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (154, 'Rik Gere', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (155, 'Busta Barrymore', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (156, 'Joshua Solido', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (157, 'Emily Goodman', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (158, 'Suzanne Capshaw', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (159, 'Jonathan Ledger', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (160, 'Olga Biggs', 'Music', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (161, 'Wayman Piven', 'Chemistry', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (162, 'Gary Stampley', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (163, 'Lynette Hanley', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (164, 'Lena Thornton', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (165, 'Kylie Trejo', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (166, 'Geoff Ryder', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (167, 'Corey Idle', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (168, 'Billy Holden', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (169, 'Nora Neuwirth', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (170, 'Toni Baranski', 'Biology', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (171, 'Hugh Pony', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (172, 'Reese Lucien', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (173, 'Sammy Englund', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (174, 'Burt McCready', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (175, 'Aida Isaacs', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (176, 'Nastassja Albright', 'Chemistry', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (177, 'Vivica Stiers', 'Math', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (178, 'Adrien Chambers', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (179, 'Martha Weir', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (180, 'Loreena Elliott', 'Chinese', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (181, 'Ashton Baranski', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (182, 'Billy Chao', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (183, 'Shirley Palminteri', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (184, 'Collin Koyana', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (185, 'Collective Willis', 'Geography ', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (186, 'Gord Osborne', 'Chemistry', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (187, 'Lindsay Amos', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (188, 'Gil Harary', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (189, 'Norm Finney', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (190, 'Lauren Guest', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (191, 'Tramaine Forster', 'Engilsh', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (192, 'Corey Doucette', 'Biology', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (193, 'Kay Blades', 'Music', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (194, 'Embeth Hidalgo', 'Chinese', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (195, 'Rachid Schiavelli', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (196, 'Hikaru Swinton', 'Chemistry', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (197, 'Benjamin Pacino', 'Engilsh', 'M');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (198, 'Edwin Holmes', 'Math', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (199, 'Loren Loggia', 'Geography ', 'F');

insert into HSNC.TEACHERS (TID, TNAME, TITLE, SEX)
values (200, 'Dustin Kline', 'Math', 'M');

commit;

```

</details>

