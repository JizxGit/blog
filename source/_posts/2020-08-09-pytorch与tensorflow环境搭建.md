---
title: pytorch与tensorflow环境搭建
tags:
  - null
categories:
  - null
date: 2020-08-09 23:21:01
---
<Excerpt in index | 首页摘要> 

记录pytorch与tensorflow的环境搭建

<!-- more -->
<The rest of contents | 余下全文>



### conda安装pytorch

1、添加conda的镜像源

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
conda config --set show_channel_urls yes#下载时显示文件来源
```

2、然后去pytorch官网，找到它给的命令，去掉-c后面部分，（它指定默认地址下属路径，若不删掉，他就又去default地址找资源了）

 **conda install pytorch torchvision** ~~-c pytorch~~

![image-20200809232604388](./pytorch安装命令.png)



### tensorflow安装

直接conda安装即可

```bash
conda install tensorflow
```

如果使用pip的话

```bash
pip install tensorflow==2.3 -i https://pypi.tuna.tsinghua.edu.cn/simple
```



### Huggingface安装



```

```



pip install tokenizers  -i https://pypi.tuna.tsinghua.edu.cn/simple

