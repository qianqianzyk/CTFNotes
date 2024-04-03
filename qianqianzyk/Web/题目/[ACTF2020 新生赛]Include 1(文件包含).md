1. 启动并访问靶机，主页只有一个tips连接，我们访问该链接，发现通过file伪协议，打开了flag.php这个文件，说明存在文件包含漏洞。
2. 既然该文件名为flag.php，那么flag应该就存在于此文件中，但是我们f12并没有查看到flag，猜测flag应该是在flag.php的源代码当中
3. 我们可以利用php://filter伪协议来查看flag.php的源代码，构造payload：?file=php://filter/convert.base64-encode/resource=flag.php 
4. 成功获取到flag.php加密后到源代码内容
5. 将其base64解密后，成功获取到flag