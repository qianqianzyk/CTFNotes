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

