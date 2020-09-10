---
title: anaconda与jupyter的使用和设置
tags:
  - python
  - anaconda
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



# anaconda 

## 安装

1. [清华镜像站下载](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/?C=M&O=D) 或者[清华镜像站anaconda首页](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/) 或者[anaconda官网](https://www.anaconda.com/distribution/)(比较慢)

    下面是每个anaconda对应的Python版本图

    ![anaconda版本记录](./anaconda版本记录.png)

2. 对于 **windows 安装**时，把Anaconda加入环境变量，这涉及到能否直接在cmd中使用conda、jupyter、ipython等命令，推荐打勾。如果没有打钩，可在后续加入下面这些路径（如果目录不同，请修改为对应的安装目录）：

    ```
    C:\ProgramData\Anaconda3;
    C:\ProgramData\Anaconda3\Library\mingw-w64\bin;
    C:\ProgramData\Anaconda3\Library\usr\bin;
    C:\ProgramData\Anaconda3\Library\bin;
    C:\ProgramData\Anaconda3\Scripts;
    ```

3. 配置镜像地址，否则从官方网站下载、升级文件太慢

    ```bash
    conda config --show channels   				# 列出现有的镜像频道
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/ # 为pytorch而加
    conda config --set show_channel_urls yes   # 下载时显示文件来源
    conda config --remove channels 镜像链接     # 删除指定的镜像
    conda clean- i 									  # 清除索引缓存
    ```

4. 推荐在默认环境下更新所有的包。打开 Anaconda Prompt （或者 Mac 下的终端），键入`conda upgrade —all`

## 使用

默认有一个base环境，推荐在此基础上，再创建2个环境，分别是：Python3和Python2

```
conda create -n py3 python=3.6      # 如果是 python=3 默认安装最新版
conda create -n py2 python=2.7
```

### 包管理

```bash
conda install numpy scipy pandas            # 安装多个包
conda install numpy=1.10                    # 指定所需的包版本
conda remove package_name                   # 卸载包
conda update package_name                   # 更新包
conda update --all                          # 更新环境中的所有包（这样做常常很有用）
conda list                                  # 列出已安装的包
conda search search_term                    # 如果不知道要找的包的确切名称，可以尝试进行搜索 
```

###  环境管理

```bash
# 创建、删除环境，记得替换 env_name 为自定义的环境名称
conda create -n env_name python=3.4 pandas  # 创建环境 ，-n 是指名称，后面可以跟一些想要安装的库
conda env remove -n env_name                # 删除指定的环境
conda create -n new_env --clone old_name    # 克隆环境，实现重命名

# 激活/退出环境
activate my_env                             # 激活环境 window
deactivate                                  # 退出环境 window
conda activate my_env                       # 激活环境 OSX/Linux
conda deactivate                            # 退出环境 OSX/Linux
source activate my_env                      # 激活环境 OSX/Linux(弃用)
source deactivate                           # 退出环境 OSX/Linux(弃用)

# 查看环境
conda env list                              # 列出你创建的所有环境
conda info -e                               # 列出你创建的所有环境
conda info                                  # 显示当前环境的全部相关信息

# 导出环境信息文件，利用文件信息克隆环境
conda env export > environment.yaml         # 将包保存为YAML，共享此文件，而且其他人能够用于创建和你项目相同的环境
conda env create -f environment.yaml        # 利用环境文件创建相同环境
```

>   对于不使用 conda 的用户，可以使用命令`pip freeze> pip_requirements.txt `（[详情](https://pip.pypa.io/en/stable/reference/pip_freeze/)）将一个 pip_requirements.txt 文件导出并包括在其中。

推荐 Mac/linux，在 `~/.bash_profile` 中添加别名，方便激活环境

```bash
alias activate="source activate"
alias deactivate="source deactivate"
```


参考：https://www.cnblogs.com/python2webdata/p/10034528.html



# jupyter/jupyter-lab

推荐安装jupyter的升级版**jupyter lab**： `conda install -c conda-forge jupyterlab `

激活环境后，输入`jupyter-lab`或者`jupyter notebook`就打开了当前环境的 notebook（**如果当前环境没有安装jupyter，那么会调用base的**）。

如果经常在Python3与Python2之间切换，为了不用每次都先切换环境才能使用 jupyter，可以进行一下配置，实现直接在 jupyter 打开的网页中指定环境。

## 使用anaconda的虚拟环境

1. 激活虚拟环境 `source activate 环境名称`

2. 安装 ipykernel，注意：在虚拟环境中安装 ipykernel，`conda install ipykernel`

3. 写入Jupyter 的 kernel 中，还是在该虚拟环境中，运行命令 `python -m ipykernel install --user --name 环境名称 --display-name "Python (环境名称)"`

4. 完成，打开Jupyter``jupyter-lab``或 `jupyter notebook`



## windows右键打开Jupyter\Jupyter-lab

注：Jupyter-lab类似

1. 打开 regedit，定位到`HKEY_CLASSES_ROOT\Directory\Background\shell`

2. 右键新建“项”，输入名称“jupyter”，该名称将出现在右键的菜单上

3. 然后在jupyter目录的右侧，新建一个字符串值`Icon`，设置为`%USERPROFILE%\AppData\Local\Continuum\anaconda3\Menu\jupyter.ico`，该图标将出现在右键的菜单上

4. 最后在jupyter目录下新建一个目录 “command”，点击command目录，修改右侧的值为`"C:\Windows\System32\cmd.exe" "--working-dir" "%v." "/k jupyter notebook"`

    > cmd /c start dir：会打开一个新窗口后执行dir指令，原窗口会关闭；
    >
    > cmd /k start dir：会打开一个新窗口后执行dir指令，原窗口不会关闭。

![jupyter注册表1](./jupyter注册表1.png)

![jupyter注册表2](./jupyter注册表2.png)

参考：https://blog.csdn.net/firing00/article/details/81866878
