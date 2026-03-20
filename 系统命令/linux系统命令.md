# linux常见命令

#### 1. ls：列出当前目录中的文件和子目录

#### 2. pwd：显示当前工作目录的路径

#### 3. cd：切换工作目录  `cd desktop`

#### 4. mkdir：创建新目录

#### 5. rmdir：删除空目录

```python
   rmdir test1 test2  #可以同时删除多个文件夹
```

#### 6. rm：删除文件或目录

        删除文件直接用 `rm test.txt`

        删除文件夹要用 `rm -r test`

#### 7. cp：复制文件或目录

```python
cp source_file destination
cp -r source_directory destination  # 递归复制目录及其内容
```

#### 8. mv：移动或重命名文件或目录

```python
mv old_name new_name
```

#### 9. touch：创建空文件或更新文件的时间戳

```python
touch file_name
```

#### 10. cat：连接和显示文件内容

```python
cat file_name
```

#### 11. more/less：逐页显示文本文件内容

        用`less`*查看文件内容，`:q` 退出当前 *less* 实例，`:qa` 会退出所有可能嵌套的 `less` 实例。

#### 12. head/tail：显示文件的前几行或后几行

```python
head -n 10 file_name  # 显示文件的前10行
tail -n 20 file_name  # 显示文件的后20行
```

#### 13. grep：在文件中搜索指定文本

```python
grep search_term file_name
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-01-16-14-09-12-image.png)

#### 14. ps：显示当前运行的进程

```python
ps aux
ps
```

<img src="file:///C:/Users/戴建/AppData/Roaming/marktext/images/2026-01-16-14-19-24-image.png" title="" alt="" width="640">

[Linux 命令详解：深入理解 `ps aux` —— 进程监控的基石 — geek-blogs.com](https://geek-blogs.com/blog/linux-ps-aux/)  

#### 15. kill：终止进程

```python
kill process_id  #这里的process_id就是上面的PID 如：kill 3320 
```

#### 16. ifconfig/ip：查看和配置网络接口信息

```python
ifconfig
ip addr show #建议ip命令，ifconfig内容不够全面
其他命令：
ip a      # 查看所有接口（相当于 ifconfig）
ip link   # 查看链路层信息
ip route  # 查看路由表（相当于 route -n）
```

#### 17. ping：测试与主机的连通性

```python
ping host_name_or_ip
```

#### 18. wget/curl：从网络下载文件

```python
wget URL
curl -O URL
```

#### 19. chmod：修改文件或目录的权限

chmod（change mode）

```python
chmod permissions file_name
```

Linux/Unix 的文件调用权限分为三级 : 文件所有者（Owner）、用户组（Group）、其它用户（Other Users）。

#### 20. chown：修改文件或目录的所有者

```python
chown owner:group file_name
```

#### 21. tar：用于压缩和解压文件和目录

```python
tar -czvf archive.tar.gz directory_name  # 压缩目录
tar -xzvf archive.tar.gz  # 解压文件
```

#### 22. top/htop：显示系统资源的实时使用情况和进程信息

```python
top
htop  #查看完按q退出
```

#### 23. ssh：远程登录到其他计算机

```python
ssh username@remote_ip
```

#### 24. scp：安全地将文件从本地复制到远程主机，或从远程主机复制到本地

```python
scp local_file remote_user@remote_host:/remote/directory
```

#### 25. grep：在文本中搜索匹配的行，并可以使用正则表达式进行高级搜索

```python
grep -r "pattern" /path/to/search
```

#### 36. sed：流编辑器，用于文本处理和替换

- **插入到指定行之前**：使用 *i* 命令和行号来插入文本。

- **插入到指定行之后**：使用 *a* 命令和行号来插入文本。

假设你想在一个名为 *example.txt* 的文件的第 3 行后插入一行文本 "This is an inserted line."，你可以使用以下命令：

```
`sed -i '3a This is an inserted line.' example.txt` #-i是在原文件上修改
```

 **一、sed删除文件第一行**

```
sed -i '1d' file.txt -- 删除第一行
sed -i 'nd' file.txt -- 删除第n行
sed -i '$d' file.txt -- 删除最后一行
```

 **二、sed插入数据 按行**

```
sed -i 'ni\x' file.txt -- 第n行前添加x内容（换行）
sed -i 'na\x' file.txt -- 第n行后添加x内容（换行）
sed -i '/m/i\x' file.txt -- 匹配m字符的行前面添加x内容
sed -i '/m/a\x' file.txt -- 匹配m字符的行后面添加x内容

- -i in front 前面
- -a after 后面
```

 **三、sed行尾、行首添加字符**

```
sed 's/^/HEAD&/g' file.txt -- 在每行的头添加字符"HEAD"

sed 's/$/&TAIL/g' file.txt -- 在每行的尾添加字符"TAIL"

- -- "^" 行首
- -- "$" 行尾
- -- "g" 代表每行出现的字符全部替换，在替换特定字符的场景下，便可发挥作用，否则只会替换每行的第一个出现字符，而不往后搜寻
```

注：添加了"g"之后，把每一个a都替换为xxxxx，不添加则只替换第一个出现的a

- -- 添加 " > b.txt" 则可以把文件保存为新的文件，如果想在原文件上进行修改，添加选项" -i " 即可
