# 知道密码，如已知密码是文件名

```txt
import zipfile
import os
path=r"C:\xxxx\attachment (4)\0573\0114\0653\0234\0976\0669\0540\0248\0275\0149\0028\0099\0894\0991\0414\0296\0241\0914" #这个自己把控想在哪里开始使用脚本
file="0140.zip"
def un_zip(Path,File_name): #传入一个路径和当前路径的压缩包名字，返回解压缩后的文件名字
        current_file=Path+os.sep+File_name #路径+'/'+文件名
        #new_path=''
        os.chdir(Path) #改变当前工作路径，方便添加文件夹
        
        zip_file=zipfile.ZipFile(current_file)
        #print(zip_file.namelist()[0])
        new_file=zip_file.namelist()[0] #新解压的压缩文件为新的路径名字
        
        #new_path=current_path + os.sep + new_file
        #os.mkdir(new_path) #新建一个以解压出来的压缩包为名字的文件夹
 
        #os.chdir(new_path)
        zip_file.extractall( path=Path, members=zip_file.namelist(), pwd=File_name[0:-4].encode() )#因为密码就是文件名
        zip_file.close()
        
        return new_file
 
new=file
new1=''
while (1):
        #new1=un_zip(path,new) #第一次解压出来了new1
        if(new ==''):  #判断是否解压完毕，是则直接退出
                print("end:"+new1)
                break
 
        else:   #否则就开始新的解压
                new1=un_zip(path,new)
                print("continue:"+new1)
                new=new1
                
        
```

# rust和john破解多层zip

```txt
cargo new solve
```

这是 Rust 语言的包管理器 Cargo 创建一个新的 Rust 项目，名为 solve

## john

```txt
./configure --enable-rexgen
```

这是在编译一个名为 john 的程序时使用的命令。configure 脚本通常用于配置软件包的安装过程。在这里，通过 --enable-rexgen 选项，启用了正则表达式的支持
进入 John 程序的源代码目录，然后执行以下命令来编译 John 程序，并启用正则表达式支持

## crack.sh

crack.sh：这是一个 Bash 脚本，用于调用 John 程序进行密码破解。它的主要作用是从一个名为 archive.zip 的压缩文件中提取哈希信息，然后使用 John 程序进行密码破解。如果找到了密码，则输出密码；如果未找到密码，则尝试使用 john --show 命令来显示所有密码，然后使用 grep 过滤出其中的密码信息。

```txt
#/bin/bash
zip2john archive.zip > zip.hashs 2>/dev/null &&
result=$(john --regex="$1" zip.hashs 2>/dev/null | grep "flag.txt" | awk '{print $1}')
if [[ -z $result ]]; then
    john --show zip.hashs 2>/dev/null | grep "flag.txt" | awk -F ":" '{print $2}'
else
    echo $result
fi
```

## main.rs

main.rs：这是一个 Rust 程序，用于解压缩 archive.zip 文件。它首先读取 archive.zip 文件，然后尝试使用密码进行解压缩。如果密码不正确，它会调用 get_password 函数来获取密码。该函数会根据压缩文件中的注释（comment）中的数字，构造一个正则表达式，然后调用 crack.sh 脚本来进行密码破解。如果密码正确，它会将解压缩后的文件重新写回 archive.zip 文件，然后继续尝试解压缩，直到成功或者不是压缩文件为止。

```txt
use std::fs::File;
use std::io::{Read, Result, Write};
use zip::ZipArchive;
use std::process::Command;

fn get_password(except_number: &u8) -> String {
    let regular_expression: String;
    if *except_number == 9 {
        regular_expression = String::from("zjut[0-8]{6}(AA|BB)");
    } else if *except_number == 0 {
        regular_expression = String::from("zjut[1-9]{6}(AA|BB)");
    } else {
        regular_expression = format!("zjut[0-{}{}-9]{{6}}(AA|BB)", except_number - 1, except_number + 1);
    }
    let output = Command::new("bash")
        .arg("crack.sh")
        .arg(&regular_expression)
        .output();
    let output = output.unwrap();
    let password = String::from_utf8(output.stdout).unwrap().trim().to_string();
    password
}

fn main() -> Result<()> {
    loop {
        let file: File = File::open("archive.zip")?;
        let mut archive = ZipArchive::new(file)?;
        let comment = archive.comment();
        let except_number = comment.last();
        if except_number.is_none() {
            // 读取文件
            let mut archive_file = archive.by_index(0).unwrap();
            let mut buffer = Vec::new();
            archive_file.read_to_end(&mut buffer)?;
            // 打印结果
            let flag = String::from_utf8(buffer).unwrap();
            println!("{}", flag);
            break; // Exit the loop if it's not a zip archive
        }
        let except_number = except_number.unwrap() - b'0';
        let password = get_password(&except_number);
        let archive_file = archive.by_index_decrypt(0, password.as_bytes());
        if archive_file.is_err() {
            drop(archive_file);
            // 读取文件
            let mut archive_file = archive.by_index(0).unwrap();
            let mut buffer = Vec::new();
            archive_file.read_to_end(&mut buffer)?;
            // 打印结果
            let flag = String::from_utf8(buffer).unwrap();
            println!("{}", flag);
            break; // Exit the loop if it's not a zip archive
        }
        let archive_file = archive_file.unwrap();
        let mut buffer = Vec::new();
        archive_file.unwrap().read_to_end(&mut buffer)?;
        let mut file = File::create("archive.zip")?;
        file.write_all(&buffer)?;
    }
    
    Ok(())
}
```

## 运行

在终端中进入到包含 main.rs 文件的目录中，然后使用以下命令来编译和运行 Rust 代码

```txt
cargo run
```

## 补充(安装cargo)

```txt
sudo apt update
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
rustc --version
cargo --version
```

![](https://s21.ax1x.com/2024/04/06/pFqa4w8.png)