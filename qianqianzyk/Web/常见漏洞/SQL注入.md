# SQL注入

## 简介
SQL 注入是一种代码注入技术，用于攻击数据驱动的应用程序。在应用程序中，如果没有做恰当的过滤，则
可能使得恶意的SQL 语句被插入输入字段中执行（例如将数据库内容转储给攻击者）。

## 分类
- 盲注
  - 布尔盲注：只能从应用返回中推断语句执行后的布尔值
  - 时间盲注：应用没有明确的回显，只能使用特定的时间函数来判断
- 报错注入：应用会显示全部或者部分的报错信息
- 堆叠注入：有的应用可以加入; 后一次执行多条语句
- 其他

## 报错注入

### SQL拼接

> 判断SQL注入闭合方式

1. 使用**转义字符**来判断SQL注入的闭合方式

    **原理**:当闭合字符遇到转义字符时，会被转义，那么没有闭合符的语句就不完整了，就会报错，通过报错信息我们就可以推断出闭合符。

    **分析报错信息**:看\斜杠后面跟着的字符，是什么字符，它的闭合字符就是什么，若是没有，就为数字型。


2. 输入1、1’、1"判断SQL语句闭合方式

   [MySQL中单引号，双引号和反引号的区别](https://blog.csdn.net/u012060033/article/details/93348261)

   一般情况下，SQL语句闭合方式为**单引号**

    > 下面以Buuctf-Web-[极客大挑战 2019]EasySQL 1为例:
   > 
   > 输入1、1"时，没有SQL语句报错，只是提示我们输入的值是不对的，因此我们可以先假设SQL语句闭合方式是单引号
   >
   >username输入1时，形成的sql语句是SELECT*FROM table_name WHERE username='1'and password='123';
   > 
   >username输入1"时，形成的sql语句是正确的
   SELECT`*`FROM table_name WHERE username='1"'and password='123';
   当字符串内需要包含双引号时，除了使用转义字符外，也可以使用一对单引号来包括字符串。字符串内的双引号被视为普通字符，无需特殊处理
   同理，当字符串内需要包含单引号时，除了使用转义字符外，也可以使用一对双引号来包括字符串。字符串内的单引号被视为普通字符，无需特殊处理
   >
   >username输入的是1'，形成的sql语句是错误的
   SELECT`*`FROM table_name WHERE username='1''and password='123';
   第一个单引号和第二个单引号形成了新的闭合，剩余第三个单引号，组成的sql语句不正确，于是语句报错。 
   > 
   >所以可以推出SQL语句闭合方式是单引号。
