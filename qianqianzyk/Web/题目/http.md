# [极客大挑战 2019]Http 1

1. 打开Burpsuite进行抓包,发现Secret.php
2. 访问/Secret.php,得到It doesn't come from 'https://Sycsecret.buuoj.cn',说明它需要我们从'https://Sycsecret.buuoj.cn'来访问它
   所以要进行伪造来绕过了

    ![](https://s21.ax1x.com/2024/04/06/pFqsUhT.png)
3. 得到Please use "Syclover" browser,说明要伪造浏览器

    ![](https://s21.ax1x.com/2024/04/06/pFqs0c4.png)
4. 得到No!!! you can only read this locally!!!,只需要改为本地ip即可

    ![](https://s21.ax1x.com/2024/04/06/pFqsgN6.png)
5. 得到flag

# [BJDCTF2020]The mystery of ip 1

## 考点
X-Forwarded-For注入

PHP可能存在Twig模版注入漏洞

## WP

结合题目名，IP的秘密，flag页面也出现了IP，猜测为X-Forwarded-For处有问题

使用BurpSuite抓取数据包

添加HTTP请求头`X-Forwarded-For: 1`

发送数据包，得到回显页面Your IP is : 1

被成功执行，说明XFF可控

>Flask可能存在Jinjia2模版注入漏洞
>PHP可能存在Twig模版注入漏洞

添加模版算式，检测其是否可被执行`X-Forwarded-For: {{7*7}}`

得到回显页面Your IP is : 49	

模版中算式被成功执行，尝试是否能执行命令`X-Forwarded-For: {{system('ls')}}`

命令可以被成功执行，查找flag的位置`X-Forwarded-For: {{system('ls /')}}`

在/目录下查找到flag，读取flag，构造payload`X-Forwarded-For: {{system('cat /flag')}}`
