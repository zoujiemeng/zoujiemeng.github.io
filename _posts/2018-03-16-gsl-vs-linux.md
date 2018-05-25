---
layout: post
title: "GSL在Windows及linux系统下的安装与使用"
tagline: ""
description: "GSL在Windows及linux系统下的安装与使用"
category: open_source_library
tags: [GSL,Windows,linux,visual studio,ubuntu]
last_updated: 2018-03-16
---

GSL(GNU Scientific Library)是用C语言写成的科学计算库，与1996年发布了0.0版，已有22年历史。该计算库全部用C语言写成，在C/C++环境下均能良好运行。由于其开源的特性，在多位开发者的贡献下，已在多个平台编译成功并且运行良好（包括使用广泛的windows和linux平台）。下面介绍GSL在上述两个平台下的安装与使用方式（目前gsl的最新版本为2.4）。

# GSL on Windows
在Windows平台上最好用的IDE无疑是Visual studio了（个人感觉没有之一）。本文以VS 2015为例，介绍gsl的安装及使用方式。（本方法使用Cmake生成VS项目文件，支持几乎所有的VS版本，尤其适合内置Cmake支持的VS2017）

## 下载Cmake
在Cmake官方网站的下载页面(https://cmake.org/download/)找到对应Windows版本的下载包，下载之后直接安装即可。

## 下载gsl源代码
在(https://github.com/ampl/gsl)下载支持Cmake的gsl最新版本zip并解压。

## 使用Cmake生成VS的工程项目文件
+ 打开Cmake-gui，选择源代码路径以及工程文件存储路径，如下图所示
![Cmake-gui]({{site.url}}/assets/images/20180316-1.png)
+ 点击configure按钮，选择对应版本的编译器（注意：VS2015的版本号是14,64位系统请选择带win64字样的选项）
![Cmake-gui-configure]({{site.url}}/assets/images/20180316-2.png)
+ configure完成后点击generate生成对应工程文件，最后点击open project即可在对应IDE中打开。

## 使用VS生成库文件
+ 编译ALL BUILD项目即可（编译过程中一些损失精度的warning可无视）。
+ 编译完成后，如果编译选项是release，即可在工程目录的release文件夹下看到`gsl.lib`和`gslblas.lib`两个库文件。

## 测试gsl库
+ 在VS中新建空项目gsl_test，并且在main.c中添加如下测试代码
```C++
#include <stdio.h>
#include <gsl/gsl_sf_bessel.h>
int main(void)
{
	double x = 5.0;
	double y = gsl_sf_bessel_J0(x);
	printf("J0(%g) = %.18e\n", x, y);
#ifdef WIN32
	system("pause");
#endif
	return 0;
}
```
+ 打开项目属性对话框，在**VC++目录->包含目录**中添加gsl源码的解压文件夹路径，在**VC++目录->库目录**中添加之前生成库文件的路径。在**链接器->输入->附加依赖项**中添加**gsl.lib;gslcblas.lib**
+ 编译项目（编译信息若提示默认库“LIBCMT”与其他库的使用冲突，只需在**项目属性->链接器->命令行**下方的输入栏中输入`/NODEFAULTLIB:"libcmt.lib" `即可）。
+ 运行项目，若显示下图结果，则代表gsl安装成功（可将上述系列设置单独存在一个设置文件中，方便移植到其他项目使用）。
![gsl_test_result]({{site.url}}/assets/images/20180316-3.png)

# GSL on Linux
本文以Ubuntu 14.04为例演示gsl在linux系统中的安装与使用，由于是从源码安装，理论上该步骤兼容所有支持gcc的linux发行版系统。

## 获取gsl源码
可在gsl官网中提供的ftp地址获取最新的gsl源码（目前最新版本为2.4）

## 解压文件
`tar -zxvf gsl-2.4.tar.gz`

## 配置编译路径并安装
`cd gsl-2.4`
`mkdir /home/jay/gsl2.4`(jay为用户名，文件夹名gsl2.4可自定义)
`./configure --prefix=/home/jay/gsl2.4`
`make`
`make check`（该步骤耗时较长，若上一步没有明显问题，可略过）
`make install`

## 更新配置文件
`vi ~/.bashrc`
在打开的文件末尾添加如下代码
`export LD_LIBRARY_PATH=/home/jay/gsl2.4/lib:$LD_LIBRARY_PATH`
`source ~/.bashrc`（使配置生效）

## 测试gsl库
+ 新建gsl_test.c文件，添加测试代码（代码和Windows平台测试时使用的一致）
+ 编译和链接
`gcc -Wall -I/home/jay/gsl2.4/include -c gsl_test.c`
`gcc -L/home/jay/gsl2.4/lib gsl_test.o -lgsl -lgslcblas -lm`
+ 运行
`./a.out`
若得到J0(5) = -1.775967713143382920e-01的结果，代表gsl安装成功。
