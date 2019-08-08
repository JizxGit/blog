---
title: jupyter使用与设置
tags:
  - python
  - jupyter
  - 右键
  - 效率
categories:
  - python
date: 2019-08-04 21:31:47
---
<Excerpt in index | 首页摘要> 

jupyter 使用介绍、与 anaconda 的虚拟环境结合 、右键打开配置

<!-- more -->
<The rest of contents | 余下全文>



### anaconda 

#### 安装



#### 使用



默认的是**base**环境，可以使用`conda env list `列出当前系统中存在多少环境，使用`activate env_name`（win）`source activate env_name` (linux) 来激活你想要使用的环境

如果是 Mac、linux，在 `~/.bash_profile` 中添加别名，方便激活环境

```bash
alias activate="source activate"
alias deactivate="source deactivate"
```



### jupyter



激活环境后，输入`jupyter notebook`就打开了当前环境的 notebook（**如果当前环境没有安装jupyter，那么会调用base的**）

为了不用每次都先切换环境才能使用 jupyter，可以进行一下配置，直接在 jupyter 打开的网页中指定环境。

#### 使用anaconda的虚拟环境

1. 激活虚拟环境 `source activate 环境名称`

2. 安装 ipykernel，注意：在虚拟环境中安装 ipykernel

    `conda install ipykernel`

3. 写入Jupyter 的 kernel中，还是在该虚拟环境中，运行命令 `python -m ipykernel install --user --name 环境名称 --display-name "Python (环境名称)"`

4. 打开Jupyter `jupyter notebook`



#### 右键打开Jupyter

1. 打开 regedit，定位到`HKEY_CLASSES_ROOT\Directory\Background\shell`

2. 右键新建“项”，输入名称“jupyter”，该名称将出现在右键的菜单上

3. 然后在jupyter目录的右侧，新建一个字符串值`Icon`，设置为

    `%USERPROFILE%\AppData\Local\Continuum\anaconda3\Menu\jupyter.ico`，该图标将出现在右键的菜单上

4. 最后在jupyter目录下新建一个目录 “command”，点击command目录，修改右侧的值为`"C:\Windows\System32\cmd.exe" "--working-dir" "%v." "/k jupyter notebook"`

    > cmd /c start dir：会打开一个新窗口后执行dir指令，原窗口会关闭；
    >
    > cmd /k start dir：会打开一个新窗口后执行dir指令，原窗口不会关闭。

![jupyter注册表1](./jupyter注册表1.png)

![jupyter注册表2](./jupyter注册表2.png)

参考：https://blog.csdn.net/firing00/article/details/81866878