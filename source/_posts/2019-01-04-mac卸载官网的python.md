---
title: mac卸载官网的python
tags:
  - python
  - 系统环境
categories:
  - python
date: 2019-01-04 14:17:11
---
<Excerpt in index | 首页摘要> 

不建议自己安装 官网的Python，因为这是典型的安装容易，删除麻烦。提供了安装器，但没有卸载器

<!-- more -->
<The rest of contents | 余下全文>



我们首先要知道其具体都安装了什么，实际上，在安装 Python 时，其自动生成:

- Python framework，即 Python 框架; 

- Python 应用目录; 

- 指向 Python 的连接。 

对于 Mac 自带的 Python，其框架目录为：`/System/Library/Frameworks/Python.framework `

而我们安装的 Python，其(默认)框架目录为：`/Library/Frameworks/Python.framework `



接下来,我们就分别(在 Mac 终端进行)删除上面所提到的三部分，其中x.x为 Python 的版本号。

第 1 步，删除框架:

`sudo rm -rf /Library/Frameworks/Python.framework/Versions/x.x `

第 2 步，删除应用目录:

`sudo rm -rf "/Applications/Python x.x" `

第 3 步，删除指向 Python 的连接:

`ls -l /usr/local/bin | grep '../Library/Frameworks/Python.framework/Versions/x.x' | awk '{print $9}' | tr -d @ | xargs rm`

第 4 步，删除环境变量$PATH



至此，我们已经成功删除 Python 的相关文件