---
title: hadoop streaming
tags:
  - hadoop streaming
categories:
  - hadoop
date: 2020-07-12 10:30:50
---
<Excerpt in index | 首页摘要> 

hadoop streaming

<!-- more -->
<The rest of contents | 余下全文>



## mapper、reducer 模板

Python3版本的 mapper、reducer 模板，一行一行的读取并处理

**注意：hadoop上的默认编码有可能不是 utf-8，因此不论python2、3都建议使用codecs模块，使用 codecs.open(file, 'r', encoding='utf-8') 是个好习惯**

如果你使用的是 **for line in sys.stdin:** 出现下面的错误怎么办呢——>**[How to prevent “UnicodeDecodeError” when reading piped input from sys.stdin?](https://stackoverflow.com/questions/53026131/how-to-prevent-unicodedecodeerror-when-reading-piped-input-from-sys-stdin/53028829)**

![非utf-8字符问题](非utf-8字符问题.png)

```python
#!/usr/bin/python
# -*- coding=utf8 -*-
"""
@Author :
@Created Time : 2020-05-21 15:00:32
@Description : TODO
"""

import sys
import logging
import codecs as cs
from sys import stdin, stdout
 
# 用于判断hadoop默认编码环境
logging.warning(stdin.encoding)
logging.warning(stdout.encoding)

def process():
    # TODO
    pass
 
def main():
    """
    main
    """
    # python3 建议这样处理编码，从标准输入读取，下面两种方式等价
    for line in cs.open(stdin.fileno(), 'r', encoding='utf-8', errors='replace'):
    #for line in cs.open(0, 'r', encoding='utf-8', errors='replace'):                                                                       
        # 1.输入
        line = line.strip('\r\n')
        cols = line.split('\t')
 
        # 2.列数校验
        if len(cols) != 2:
            logging.warning(line)
            continue
 
        # 3.TODO 具体处理逻辑
        results = process()
         
        # 4.输出
        for result in results:
            stdout.write(result)
            stdout.write('\n')
 
 
if __name__ == "__main__":
    main()
```



shell版的mapper、reducer，下面是一个统计词频的功能

**注意：头部不可以加 source ~/.bashrc ,因为hadoop环境没有该文件**

```bash
# mapper.sh
echo $LANG >&2
awk -F"\t" '{print $1"\t"1}'
 
 
# reducer.sh
echo $LANG >&2
awk -F"\t" '{dict[$1]+=1}  END{for(w in dict)print w"\t"dict[w]}'
```

## 本地测试

通过模拟MapReduce进行测试，排查问题。只有本地测试通过，才能确保在hadoop上运行成功。

**python测试**

构造一份测试数据到 test_data.txt 中，然后下载hadoop上的python包（自己编译的），进行测试

```bash
hadoop fs -get /xxx/python3.6.tar.gz .
tar -zxvf python3.6.tar.gz
 
# 注意使用解压后的python，非本地开发机的python
cat test_data.txt  | python3.6/bin/python mapper.py | python3.6/bin/python reducer.py
```

如需安装其他模块请使用 ./python3.6/bin/python -m pip install xxx ，**并重新打包压缩，上传到你的hadoop目录**

**其他测试**

mapper 与 reducer可以是 python、shell脚本，并且可以混合使用，如下：

```bash
cat test_data.txt  | python3.6/bin/python mapper.py | sh reducer.sh     # python 与 shell脚本
cat test_data.txt  | python3.6/bin/python mapper.py | sort -k1,1        # python 与 命令
```





## hadoop stream 模板

1. 如果只有2、3个代码文件，使用 -file xxx.py 指定；
2. 如果有较多文件，如下图，建议将整个src目录压缩成 **src.tar.gz**，上传到指定的hadoop目录，如果还用到别的目录，比如config、data也一起打包上传，并使用 -*cacheArchive hadoop路径*  指定；（hadoop支持zip, jar, tar.gz格式的压缩包，由于Java解压zip压缩包时会丢失文件权限信息而且遇到中文文件名会出错，所以**建议采用tar.gz压缩包**。）
   ![hadoop目录结构](./hadoop目录结构.png)
3. hadoop上的目录结构，可以理解与本地的工程目录是相同的，只要你打包与解压的方式是对的，因此相对路径啥的，只要本地测试通过，hadoop上也不会有问题；
4. 具体使用请看下面的模板；

```bash
#!/usr/bin/sh
source ~/.bashrc
 
DATE=`date -d "$1" +"%Y%m%d"`
HADOOP_DIR=""  # TODO hadoop路径
HADOOP_INPUT="${HADOOP_DIR}/input/"
HADOOP_OUTPUT="${HADOOP_DIR}/output/"
SRC="${HADOOP_DIR}/src.tar.gz"
DATA_CONFIG="${HADOOP_DIR}/data.tar.gz"
PYTHON="${HADOOP_DIR}/python3.6.tar.gz"
TASK_NAME="task_name"
 
hadoop fs -test -e $HADOOP_DIR
if [ $? -ne 0 ]; then
  echo "Hadoop路径有误：$HADOOP_DIR，请检查"   
  exit 8
fi
# 上传本地工程代码到 hadoop
function upload() {
  LOCAL_FILE=$1    # 本地文件
  HADOOP_FILE=$2   # hadoop目录
  echo -e "$1\n上传到↑=====↓ \n $2"
  hadoop fs -test -e $HADOOP_FILE
  if [ $? -eq 0 ]; then
    hadoop fs -rmr $HADOOP_FILE
  fi
  hadoop fs -put $LOCAL_FILE $HADOOP_FILE
}
 
function info() {
  echo -e "############### $1 ##############\n"
}
 
archives_dir="./archives" # 本地缓存压缩包的目录
[ ! -e ${archives_dir} ] && mkdir ${archives_dir}
 
info '[开始]更新本地代码到hadoop'
tar -czvf ${archives_dir}/src.tar.gz  src/  && upload ${archives_dir}/src.tar.gz $SRC
tar -czvf ${archives_dir}/data.tar.gz data/ && upload ${archives_dir}/data.tar.gz $DATA_CONFIG
# 针对不经常修改的、含有大文件的目录，避免每次都压缩，可以如下加个判断
#if [ ! -f ${archives_dir}/config.tar.gz ]; then # config是大文件，避免每次压缩
#    tar -czvf ${archives_dir}/config.tar.gz config/  && upload ${archives_dir}/config.tar.gz $NLPC_CONFIG
#fi
info '[完成]更新本地代码到hadoop'
 
hadoop fs -test -e $HADOOP_OUTPUT
if [ $? -eq 0 ]; then
  hadoop fs -rmr $HADOOP_OUTPUT
fi
 
# 务必使用如下的-cmdenv指定环境的编码为UTF8
hadoop streaming \
    -input $HADOOP_INPUT \
    -output $HADOOP_OUTPUT \
    -mapper "sh src/script/mapper.sh" \
    -reducer "sort -u" \
    -file xxx.py \
    -file yyy.sh \
    -cacheArchive ${DATA_CONFIG}\
    -cacheArchive ${SRC}\
    -cacheArchive ${PYTHON}\
    -cmdenv LANG=zh_CN.utf8 \
    -cmdenv PYTHONIOENCODING=utf_8 \
    -jobconf mapred.job.name=$TASK_NAME \
    -jobconf mapred.job.map.capacity=400 \
    -jobconf mapred.map.tasks=100 \
    -jobconf mapred.reduce.tasks=100 \
    -jobconf stream.memory.limit=5120 \
    -jobconf abaci.split.optimize.enable=false \
    -jobconf mapred.map.over.capacity.allowed=false \
    -jobconf mapred.job.priority=NORMAL \
    -jobconf abaci.job.base.environment=centos6u3_hadoop
  
if [ $? -ne 0 ]
    then
    echo "hadoop error"
    exit 1
fi
```



## 参数说明

**输入输出文件**

`-input <path>`：指定作业输入，path可以是文件或者目录，可以使用\*通配符，-input选项可以使用多次指定多个文件或目录作为输入。

`-output <path>`：指定作业输出目录，path必须不存在，而且执行作业的用户必须有创建该目录的权限，-output只能使用一次。

**上传 单个mapper、reducer文件**

-file xxx.py 

-file yyy.sh

**上传较多代码文件**

建议本地打包后上传到hadoop上，并使用 `-cacheArchive` 指定，该命令会自动解压；**注意第3个有#号的例子，这不是注释，而是指定了解压的目录，相当于 `tar -xzvf python.tar.gz -C ./target_dir`**

```
-cacheArchive ${DATA_CONFIG}\
-cacheArchive ${SRC}\
-cacheArchive ${PYTHON}#target_dir\
```

**解决reduce积压到一个reducer的问题**

 -jobconf stream.map.output.field.separator=.      # 指定“.”作为map输出内容的分隔符，默认是\t，**这个参数不建议添加和修改**
 -jobconf stream.num.map.output.key.fields=2     # 从在第2个“\t”之前的部分作为key，之后的部分作为value

**自己指定mapper的数量，否则abaci会自动优化**

```
-jobconf abaci.split.optimize.enable=false 
-jobconf mapred.map.tasks=200
```

但并不是你指定多少就是多少mapper

**指定reducer数量**

```
-jobconf mapred.reduce.tasks=100
```

**同时运行的mapper、reducer数量限制**

```
-jobconf mapred.job.map.capacity=200 最多同时运行map任务数
-jobconf mapred.job.reduce.capacity=200 最多同时运行reduce任务数
```

**设置环境变量**

`-cmdenv NAME=VALUE`：给mapper和reducer程序传递额外的环境变量，NAME是变量名，VALUE是变量值。

```
-cmdenv LANG=zh_CN.utf8 \
-cmdenv PYTHONIOENCODING=utf_8 \    # 如果上面的环境变量不生效，再试试加上这个
```

可以设置的编码请用: **locale -a |grep zh** 查看，原因请看：[Python3.6 sys.stdout.encoding的输出为ANSI_X3.4-1968(python3.6的坑)](https://blog.csdn.net/j___t/article/details/97705231)



## 官方教程

官方教程：[Hadoop Streaming](http://hadoop.apache.org/docs/r1.0.4/cn/streaming.html)