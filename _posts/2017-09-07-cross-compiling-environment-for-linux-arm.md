---
layout: post
title: "Ubuntu-14.04下arm与gcc交叉编译环境的搭建与应用"
tagline: ""
description: "linux-14.04下arm与gcc交叉编译环境的搭建与应用"
category: linux
tags: [linux,cross compile,arm,gcc]
last_updated: 2017-09-14
---

在linux下搭建起arm开发板与gcc交叉编译的环境后，可以很方便的进行嵌入式开发。借助Samba和SecureCRT还可实现在windows下的开发，非常方便。

+ 准备好linux环境（本例由于特殊情况限制使用的是Ubuntu 14.04，理论上 Ubuntu 16.04以及其他Linux内核的系统都是可以使用的，[Ubuntu 16.04的安装和前期准备工作]({% post_url 2017-03-30-vmware-ubuntu-16.04 %})在本博客里面也有所介绍）

+ 按照目标开发板的要求安装交叉编译器，以飞凌开发板为例

  + 步骤1：将文件gcc-4.6.2-glibc-2.13-linaro-multilib-2011.12.tar.gz拷贝到Ubuntu主目录下
    ` mx6用户光盘(B)\工具\gcc-4.6.2-glibc-2.13-linaro-multilib-2011.12.tar.gz`

  + 步骤2：在Ubuntu中新建一个终端，输入下面的命令安装交叉编译器：

  ```
  su (先获取root权限)
  cd (进入主目录) 
  mkdir -p /opt/freescale/usr/local (创建目录，若目录已存在会提示错误，跳过即可） 
  tar zxvf gcc-4.6.2-glibc-2.13-linaro-multilib-2011.12.tar.gz -C opt/freescale/usr/local （编译器解压到/opt/freescale/usr/local ）
  ```

  + 步骤3：查看gcc-4.6.2-glibc-2.13-linaro-multilib-2011.12是否解压成功
    `ls -l /opt/freescale/usr/local`
    如可以看到gcc-4.6.2-glibc-2.13-linaro-multilib-2011.12文件夹即解压成功
  + 步骤4：添加环境变量到Profile，执行命令打开编辑Profile
    `gedit /etc/profile `
    最后一行添加以下内容 
  ```
  export ARCH=arm
  export CROSS_COMPILE=/opt/freescale/usr/local/gcc-4.6.2-glibc-2.13-linaro-multilib-2011.12/fsl-linaro-toolchain/bin/arm-none-linux-gnueabi- 
  export PATH=/opt/freescale/usr/local/gcc-4.6.2-glibc-2.13-linaro-multilib-2011.12/fsl-linaro-toolchain/bin:$PATH 
  ```
  ​	之后执行命令： 
  ​	`source /etc/profile `
  ​	来使得刚配置的环境变量生效。
  + 步骤5： 在终端里面执行以下命令，验证交叉编译器安装是否成功 
    `arm-fsl-linux-gnueabi-gcc -v `
    安装成功后会提示COLLET_GCC、target等信息，之后就可以使用该编译器来编译Uboot 代码和内核代码了. 注意：1. Ubuntu64位版本的话运行以上命令如果提示`No such file or directory`的话，说明系统缺少lib32ncurses5库，使用apt-get安装后即可解决问题，该问题来源是目标交叉编译器是32位的。2. 以上操作均是以root 用户登录系统操作为例； 所修改的文件仅对当前用户有效， 如果通过终端切换用户，以上修改的文件对新用户无效。

+ 安装依赖包
  Linux系统的编译需要安装一些工具包，可执行开发板公司提供的脚本进行自动安装。本节操作前必须确保您的计算机或虚拟机能正常连接互联网，如您在安装中出现网络断开连接请再按照以下步骤进行安装。
  + 将文件[setup_env.sh]({{site.url}}/assets/other/setup_env.sh)拷贝至Ubuntu主目录下
  + 给setup_env.sh添加可执行权限： `chmod u+x setup_env.sh`
  + 执行脚本： `./setup_env.sh`
  + 安装过程中出现需手动确认的情况确认即可

+ 以上步骤都完成之后就可用arm-fsl-linux-gnueabi-gcc编译器进行编译了，根据[对应的代码]({{site.url}}/assets/other/0908.zip)，makefile文件可如下方式写成（将CC值改成gcc可在Linux里面运行验证）
```
CC = arm-fsl-linux-gnueabi-gcc

bk_alg_test_0906 : main.o BK_wbc_main.o impedance.o pulse_identify.o
	$(CC) -o bk_alg_test_0906 main.o BK_wbc_main.o impedance.o \
	       pulse_identify.o -lm
main.o : main.c pulse_identify.h impedance.h alg.h BK_wbc_main.h
	$(CC) -c main.c
BK_wbc_main.o : BK_wbc_main.c impedance.h BK_wbc_main.h
	$(CC) -c BK_wbc_main.c
impedance.o : impedance.c datatype.h impedance.h alg.h p2o_def.h
	$(CC) -c impedance.c
pulse_identify.o : pulse_identify.c datatype.h pulse_identify.h p2o_def.h
	$(CC) -c pulse_identify.c
clean :	
	rm bk_alg_test_0906 main.o BK_wbc_main.o impedance.o pulse_identify.o
```


+ 之后通过串口连接开发板，得到串口号后通过SecureCRT打开端口，将编译生成的文件及运行过程中用到的相关文件下载至开发板运行即可，具体步骤如下
  + 利用命令`udhcpc -i eth0`自动获得电脑IP
  + 利用下载命令`busybox tftp -g -r data_report.ko(下载文件名) 192.168.29.107(主机IP)`将需要的文件下载至开发板
  + 可利用上传命令`busybox tftp -p -l Rbc(上传文件名) 192.168.29.95(主机IP)`将应用生成文件上传至主机
