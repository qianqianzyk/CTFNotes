# [RoarCTF 2019]Easy Java 1

## 前置知识

### WEB-INF/web.xml泄露

![](https://s21.ax1x.com/2024/04/17/pFzeyxf.png)

### java web工程目录结构

![](https://s21.ax1x.com/2024/04/17/pFzefaj.png)

### Servlet访问URL映射配置

>由于客户端是通过URL地址访问Web服务器中的资源，所以Servlet程序若想被外界访问，必须把Servlet程序映射到一个URL地址上
>这个工作在web.xml文件中使用<servlet>元素和<servlet-mapping>元素完成
><servlet>元素用于注册Servlet，它包含有两个主要的子元素：<servlet-name>和<servlet-class>，分别用于设置Servlet的注册名称和Servlet的完整类名
>一个<servlet-mapping>元素用于映射一个已注册的Servlet的一个对外访问路径，它包含有两个子元素：<servlet-name>和<url-pattern>，分别用于指定Servlet的注册名称和Servlet的对外访问路径。例如：
```
<servlet>
    <servlet-name>ServletDemo1</servlet-name>
    <servlet-class>cn.itcast.ServletDemo1</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>ServletDemo1</servlet-name>
    <url-pattern>/ServletDemo1</url-pattern>
</servlet-mapping>
```

## WP

![](https://s21.ax1x.com/2024/04/17/pFzmCQK.png)
```
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" version="4.0">
<welcome-file-list>
<welcome-file>Index</welcome-file>
</welcome-file-list>
<servlet>
<servlet-name>IndexController</servlet-name>
<servlet-class>com.wm.ctf.IndexController</servlet-class>
</servlet>
<servlet-mapping>
<servlet-name>IndexController</servlet-name>
<url-pattern>/Index</url-pattern>
</servlet-mapping>
<servlet>
<servlet-name>LoginController</servlet-name>
<servlet-class>com.wm.ctf.LoginController</servlet-class>
</servlet>
<servlet-mapping>
<servlet-name>LoginController</servlet-name>
<url-pattern>/Login</url-pattern>
</servlet-mapping>
<servlet>
<servlet-name>DownloadController</servlet-name>
<servlet-class>com.wm.ctf.DownloadController</servlet-class>
</servlet>
<servlet-mapping>
<servlet-name>DownloadController</servlet-name>
<url-pattern>/Download</url-pattern>
</servlet-mapping>
<servlet>
<servlet-name>FlagController</servlet-name>
<servlet-class>com.wm.ctf.FlagController</servlet-class>
</servlet>
<servlet-mapping>
<servlet-name>FlagController</servlet-name>
<url-pattern>/Flag</url-pattern>
</servlet-mapping>
</web-app>
```

这里看到FlagController,构造其的访问路径

由上面前置知识可知，通过url访问Servlet的方式是：

找到对应文件名，然后通过这个文件名找到对应的servlet，再通过这个servlet的文件名，获取到其具体的servlet文件。因为这个是类中的文件，所以后缀要加.class

```WEB-INF/classes/com/wm/ctf/FlagController.class```

得到base64编码，解码得到flag
