# windows系统命令

## 1. 系统信息命令

winver  查看Windows版本。

systeminfo :查看系统详细信息（补丁、版本、硬件信息

hostname :显示计算机主机名

net time



## 2.网络工具

ping  :检测网络连通性，验证目标IP是否可以访问。 ping www.baidu.com

tracert :跟踪数据包到达目标主机的路径。  tracert  www.microsoft.com

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-01-14-23-48-03-image.png)

nslookup：查看域名信息，检查DNS解析是否正确。

netstat：查看当前网络连接状态，显示活动的网络连接和端口。

route print：查看路由表。

route add：添加路由。

route delete：删除路由。

netsh interface show interface：查看网络适配器的状态和配置。

netsh interface ip show address：显示IP地址信息。

netsh interface ip show route：显示路由信息。

ipconfig：查看网络配置。

ipconfig /all：查看详细的网络配置，包括MAC地址和DNS服务器。

ipconfig /flushdns：清除DNS缓存，解决DNS解析问题。

ipconfig /release：释放当前IP地址。

ipconfig /renew：重新获取IP地址。

## 3. 计算机管理命令

regedit：打开注册表编辑器，用于查看和修改Windows的注册表项。

tasklist：列出当前系统中的所有运行进程。

taskkill：终止指定的进程，常用于杀死恶意进程。

msconfig：配置启动项，启用或禁用启动程序。

net user：查看当前用户信息。

net user username /add：添加新用户。

net user username /del：删除用户。

net localgroup：查看本地用户组。

net localgroup groupname /add：添加用户至指定用户组。

net localgroup groupname /del：从用户组中删除用户。



## 4. 文件和目录操作

cd：切换目录。

dir：显示当前目录的文件和子目录。

mkdir (或者使用md)：创建目录。

del (erase)：删除文件。

copy：复制文件或目录。

move：移动文件或重命名文件和目录。

ren (rename)：重命名文件或目录。

attrib：显示或更改文件属性，如只读、隐藏等。

type：显示文本文件的内容。

echo：在命令行显示消息或将数据输出到文件。 

> 如果我要创建一个1.txt文件，# 先进入test文件夹（如果存在）
> 
> cd test
> 
> 创建并写入1.txt文件
> 
> echo 这是文件内容 > 1.txt
> 
> 或者追加内容（不会覆盖原有内容）
> 
> echo 这是追加的内容 >> 1.txt

find：查找文件中的特定文本内容。

> find [路径] [匹配条件] [动作]
> 
> 查找当前目录下名为 *file.txt* 的文件： find . -name file.txt



## 5. 安全与权限管理

whoami：查看当前登录的用户。

net user username /active：no 禁用用户账户。

net user username /active：yes 启用用户账户。

secpol.msc：打开本地安全策略管理工具。



## 6. 远程访问与控制

mstsc：启动远程桌面连接。

query user：查询当前用户信息。

shutdown /r /f /t 0：立即重启计算机。

shutdown /s /f /t 0：立即关机。
