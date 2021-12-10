---
tags:
  - Oracle
mathjax: true
comments: true
hidden: false
date: 2021-12-10 15:53:06
title: Oracle EXISTS用法
---
***
​		昨天在写一段Oracle SQL的时候，本来想直接用SQL写的，后面遇到了点问题，不了解该怎么写下去了，于是就换成了用存储过程来写，搞了个游标变量，然后搞了个循环搞定了，今天导师指导的时候看到了，指出了问题。<!-- more -->
***
## SQL EXISTS

SQL是关于集合的，要以面向集合的思维方式来思考。在编写代码时，我似乎一点也没想到这个，搞一个变量，然后开一个循环开始按记录来修改。还是要转变一下思路，从集合角度考虑。

EXCISTS用法直接写例子吧：

```sql
select * from t1 where exists ( select null from t2 where y = x )
```

这里的用法相当于运用循环控制结构后的如下语句：

```plsql
for x in ( select * from t1 )    
loop       
	if ( exists ( select null from t2 where y = x.x )       	then           
    	OUTPUT THE RECORD       
end if    
end loop
```

思路有点奇特