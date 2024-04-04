# 创建python2环境

## 下载conda

```kali-linux
wget https://repo.anaconda.com/miniconda/Miniconda2-latest-Linux-x86_64.sh
```

## 安装

```kali-linux
bash Miniconda2-latest-Linux-x86_64.sh
```

## 重新加载配置文件

```kali-linux
source ~/.bashrc
```

## 创建环境

```kali-linux
conda create --name python2env python=2
```

## 激活环境

```kali-linux
conda activate python2env
```

## 运行脚本

注意:如果缺少相应的库，正常安装即可
```kali-linux
python2 你的脚本路径
```

## 退出当前环境

```kali-linux
conda deactivate
```
