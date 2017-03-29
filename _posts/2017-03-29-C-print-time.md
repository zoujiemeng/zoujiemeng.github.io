---
layout: post
title: "C语言编程中使用time.h及宏定义打印特定代码段的运行时间"
tagline: ""
description: "基础C语言"
category: C
tags: [C,time,macro]
last_updated: 2017-03-29
---

在C语言程序中，有时候需要测试特定代码段的运行时间，下面的代码通过添加头文件“time.h”和宏定义实现debug模式下的代码段运行时间检测。




	#include "time.h" 
	#ifdef _DEBUG_MODE_ 
	  clock_t tmStart,tmEnd; 
	  long mSec; 
	#endif 
	#ifdef _DEBUG_MODE 
	  tmStart = clock(); 
	#endif 
	  特定代码段 
	#ifdef _DEBUG_MODE_ 
	  tmEnd = clock(); 
	  mSec = (tmEnd-tmStart) ; // CLOCKS_PER_SEC 
	  printf("calculate time:%ld mSec\n",mSec); 
	#endif 

