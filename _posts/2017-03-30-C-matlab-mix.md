---
layout: post
title: "C/C++与MATLAB的混合编程"
tagline: "C++"
description: "C/C++与MATLAB的混合编程"
category: mix
tags: [C++,MATLAB,mexFunction]
last_updated: 2017-03-30
---
MATLAB可以通过mexFunction作为一个中转函数，来调用C/C++的代码。并且通过VS可以实现联合调试，对于算法的可视化分析来说非常方便。
## mexFunction函数的编写
1. 包含头文件`#include "mex.h"`
2. 函数申明（该函数申明的名称，变量类型，变量名称都不可更改）
```C++
//**********************************************************************
// 函数名称: mexFunction     
// 函数说明：matlab测试调用函数     
// 返 回 值: void	     
// 参    数: 
//           [out] int nlhs			输出参数个数
//           [out] mxArray * plhs[]	输出参数matlab特殊数组
//           [in] int nrhs			输入参数个数
//           [in] mxArray * prhs[]	输入参数matlab特殊数组
//**********************************************************************
void mexFunction(int nlhs,mxArray* plhs[],int nrhs, const mxArray* prhs[])
{
	if(nlhs != 1) mexErrMsgTxt("1 outputs required");
	if(nrhs != 2) mexErrMsgTxt("2 inputs required");
}
```
3. 输入参数转化
该函数输入参数为MATLAB矩阵，需转换为C/C++对应的变量类型
+ 获取标量：函数`mxGetScalar`，该函数返回标量，直接强制类型转换即可使用
+ 获取矩阵：函数`mxGetData`，该函数返回void指针，之后按变量类型转换即可
+ 获取字符串
      char *input_buf;
      input_buf = mxArrayToString(prhs[0]);//将第一个输入参数转换为c、c++字符串
+ 获取结构体成员变量值：`mxGetField(prhs[0],0,"变量名称")`
4. 输出参数转化
在mexFunction中需将C/C++的变量转换为MATLAB的矩阵变量，这样MATLAB才能获得变量的值。
+ 申明指针`int * pDataOut;`
+ 创建矩阵`plhs[0] = mxCreateNumericMatrix(nDataLen,1, mxINT32_CLASS, mxREAL);`
+ 关联指针`pDataOut = (int *)mxGetData(plhs[0]);`
+ 修改指针的值即可（PS：接收指针无需申请内存空间，mxCreateNumericMatrix函数已申请）

## MATLAB与Visual Studio的联合调试
1. 新建空项目
2. 添加相关源文件，并在(ProjectName).c/cpp文件中编写MEX函数。
3. 打开项目属性页
  + 配置管理器：选择对应环境（32位或64位）
  + 常规->配置类型->动态库
  + 常规->目标文件扩展名->.mexw64(32位环境改为.mexw32)
  + C++->附加包含目录：添加MATLAB安装目录下的\extern\include和\extern\include\win64路径（32位按对应环境更改）
  + 链接器->输出文件改为$(OutDir)$(TargetName)$(TargetExt)
  + 链接器->附加库目录添加MATLAB安装目录下的\extern\lib\win64\microsoft路径（32位按对应环境更改）
  + 链接器->输入->附加依赖项添加下列四个库文件
  > libmx.lib
  > libeng.lib
  > libmat.lib
  > libmex.lib

4. Source Files->Add->New Item新建模块定义文件(ProjectName).def，并添加如下内容
> LIBRARY;"(ProjectName)"
> EXPORTS mexFunction

然后打开项目属性对话框，在Linker-Input-Module Definition File添加：(ProjectName).def
5. 编译生成。查看目标输出目录是否有(ProjectName).mexw64文件（32位系统同理）
6. 将matlab的current folder 设置成mexw64文件所在的路径,或者移动生成的mexw64文件到MATLAB的current folder。
7. VS中选择调试->附加到进程->MATLAB
8. 断点调试即可
（ps：每次修改MexFunction所在的.c文件后，重新编译生成解决方案前都需要先在matlab工程下clear一下，即`clear (ProjectName).mexw64`）


