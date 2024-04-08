1. 将流量包下载下来之后，发现文件大小异常，猜测文件中包含其他文件
2. 使用binwalk或foremost分离,发现有个zip文件,并且需要密码
3. 发现一句话“这可是administrator的秘密，怎么能随便给人看呢？”，administrator是windows操作系统管理员用户，猜测和用户密码有关。

    重新分析流量包，搜索查看常用协议，发现在http协议中存在lsass.dmp

    （*dmp文件是windows系统中的错误转储文件，当Windows发生错误蓝屏的时候，系统将当前内存【含虚拟内存】中的数据直接写到文件中去，方便定位故障原因。）

    （*里面包含主机用户密码信息）
4. 将lsass.dmp提取出来,使用mimikatz分析.dmp文件
5. 在kali-linux中输入mimikatz,查找.exe文件,找到后以管理员的身份运行(在Windows下,注意关掉火绒等安全软件防火墙)
6. //提升权限 

   privilege::debug

   //载入dmp文件

   sekurlsa::minidump lsass.dmp

   //读取登陆密码

   sekurlsa::logonpasswords full
7. 找到Administrator下的密码,解出zip从而得到flag