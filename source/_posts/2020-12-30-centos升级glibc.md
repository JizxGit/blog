---
title: centos升级glibc
tags:
  - glibc
categories:
  - linux
date: 2020-12-30 23:23:31
---
<Excerpt in index | 首页摘要> 

<!-- more -->
<The rest of contents | 余下全文>



## 风险提醒

- GLIBC是非常多命令的底层库，一旦升级失败，将面临重装系统的风险；

- 如果有重要数据，请提前备份；

- 请开3个连接窗口，一个root用户（实际升级操作）、一个普通用户、一个root用户，另外两个是在失败时进行补救的窗口；

  

## 查看本机GLIBC版本

```
strings /lib64/libc.so.6 | grep GLIBC_
```



## 开始升级

有两种升级方式：1、rpm（推荐），2、源码编译

### 方法一、 rpm 升级glibc-2.17

```bash
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-2.17-55.el6.x86_64.rpm
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-common-2.17-55.el6.x86_64.rpm
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-devel-2.17-55.el6.x86_64.rpm
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-headers-2.17-55.el6.x86_64.rpm
 
su root   # 一定要用root ，否则失败无法回滚

rpm -Uvh glibc-2.17-55.el6.x86_64.rpm \
glibc-common-2.17-55.el6.x86_64.rpm \
glibc-devel-2.17-55.el6.x86_64.rpm \
glibc-headers-2.17-55.el6.x86_64.rpm --force --nodeps
```



### 方法二、源码编译升级glibc-2.14

1. 下载glibc-2.14源码：[glibc-2.14.tar.gz](http://ftp.gnu.org/gnu/glibc/glibc-2.14.tar.gz)，其他版本可以在下载目录：http://ftp.gnu.org/gnu/glibc/ 中找到

2. `su root`  # **一定要用root** ，否则失败无法回滚

3. 编译，安装，设置软链接

   ```bash
   su root    # 一定要用root
   tar -xzvf glibc-2.14.tar.gz
   cd glibc-2.14
   mkdir build && cd build
   ../configure --prefix=/opt/glibc-2.14
   # 上面这种不行的话再试试下面
   # ../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
   make -j4 && make install   # 如果想并行，可以指定-j，数量为机器的cpu核数，如4核，即make -j4，八核即make -j8 如果串行的话，就直接make
     
   # 重新设置软链接
   rm -rf /lib64/libc.so.6 && LD_PRELOAD=/opt/glibc-2.14/lib/libc-2.14.so ln -s /opt/glibc-2.14/lib/libc-2.14.so /lib64/libc.so.6
   ```

4. 在执行 `rm -rf /lib64/libc.so` 后可能会导致系统命令不可用，应该立刻**回滚到之前的版本**

   `LD_PRELOAD=/lib64/libc-2.12.so ln -sf /lib64/libc-2.12.so /lib64/libc.so.6 `  # f表示强制

5. 检查是否成功

   - 如果安装成功，那么运行ls命令，应该能直接显示该目录下的文件，如果有报错，类似“ls: error while loading shared libraries: __vdso_time: invalid mode for dlopen(): Invalid argument”，这就说明有错误了，安装失败。可使用步骤1的恢复命令，进行恢复。一旦安装失败，请不要断开ssh，否则再也无法登录进去了。
   - `strings /lib64/libc.so.6 | grep GLIB`，显示了2.17即表示安装成功了。

6. 经测，这种方式在centos6.3系统升级glibc-2.17不成功，glibc-2.17建议使用rpm