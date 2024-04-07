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