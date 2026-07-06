# Anaconda 基本使用

## 一、安装
**下载：**[https://mirror.tuna.tsinghua.edu.cn/help/anaconda//](https://mirror.tuna.tsinghua.edu.cn/help/anaconda//)



**配置国内源：**

```plain
channels:
  - defaults
show_channel_urls: true
default_channels:
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch-lts: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
ssl_verify: true
```

## 二、常用命令
### 1、环境管理命令
```bash
conda info -e
```

```bash
conda env list 
```

```bash
conda activate python27 
```

```bash
conda remove --name python34 --all
```

```bash
conda create --name py37 python=3.7
```



### 2、包管理命令
```bash
# 记得要先激活对应环境
conda install tensorflow 
```

```bash
conda list
```

```bash
conda upgrade --all
```



> 更新: 2023-04-16 00:50:27  
> 原文: <https://www.yuque.com/thinkspace/cn8pmn/ofga6s>