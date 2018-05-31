---
layout: post
title: "VS Code的windows安装及语言支持(C/C++,Python,Markdown)"
tagline: ""
description: "VS Code的安装及语言支持(C/C++,Python,Markdown)"
category: vscode
tags: [vscode,C,C++,python,markdown]
last_updated: 2018-05-31
---

当我们只想做一些轻量级项目或者是只需要一个跨平台的代码编辑器时，Codeblock似乎是一个可行的选择。然而，在8102年的今天，更年轻的VS Code似乎是一个更Coooool的选择。微软从来就不缺人才，而当微软开始拥抱开源的时候，我们有理由抱有更高的期待，相信他们能够做出一款优秀的跨平台编辑器。本文介绍截止至文章更新日，VS Code的windows安装以及部分语言支持(C/C++,Python,Markdown)。

## VS Code在windows环境下的安装
找到[VS Code的官方网站](https://code.visualstudio.com/)，点击`download for windows`，下载完后点击exe安装即可，这个过程非常简便，在此不再赘述。

## C/C++的环境配置
安装完后，VS Code还只是一个简单的文字编辑器，只是多了一些就简单的附加功能，例如对git和markdown的支持。并不支持C/C++代码的编译、查找、debug等功能，要想实现上述功能，需要对其进行进一步的配置。

### 安装C/C++插件
打开vscode，最左侧sidebar中点击扩展(Extensions)，查找C++插件，下载安装排在第一个的插件，并点击reload。（注意：该插件本身并未包含任何C/C++的编译器，这正好给了我们选择的空间，用自己喜爱的编译器，本文以mingw-w64编译器为例）

### 安装喜爱的编译器(本文以mingw-w64编译器为例)
+ 找到[mingw-w64的公共代码仓库](https://sourceforge.net/projects/mingw-w64/files/)，点击`Download Latest Version`，就能下载到一个在线安装的exe，所谓在线安装，就是在安装过程中根据安装选项自动下载对应的文件，但是由于网络问题，exe的下载可能比较慢，因此在了解安装选项的情况下，可以到上述代码仓库自行下载对应的.7z文件，解压至目标目录即可。例如，64位的seh、POSIX线程的最新稳定版本的gcc包的7z文件就是在上述网页往下翻，点击`x86_64-posix-seh`的链接即可获得。
+ 编译器安装完成后，需要配置系统变量：`右击计算机--属性--高级系统设置--环境变量--双击path变量--添加mingw-w64安装目录下bin文件夹的路径`
+ 配置好系统变量后，打开命令行，输入`gcc --version`查看输出，正常显示版本信息则代表安装成功。

### 配置Intellisense
随便新建或者打开一个.c或者.cpp，输入简单的代码，例如
```C++
#include <stdio.h>

void main()
{
    printf("hello world!\n");
}
```
可能会提示找不到对应的头文件。此时点击快捷键`Ctrl+Shift+P`打开命令行，点击`C/Cpp: Edit configurations...`，此时会在源代码文件夹下的.vscode文件夹（该文件夹下保存vscode需要的配置文件）下生成`c_cpp_properties.json`文件，该文件定义了项目的编译器路径，包含目录等内容，如果是使用mingw作为编译器的话，只需要如下设置即可（该设置会自动搜寻并设置对应的标准库头文件对应的目录，并不需要像网上所说的那样将所有mingw的include文件夹加进去）。
```json
{
    "configurations": [
        {
            "name": "MinGW",
            "intelliSenseMode": "clang-x64",
            "compilerPath": "C:/Program Files/mingw-w64/x86_64-8.1.0-release-posix-seh-rt_v6-rev0/mingw64/bin/gcc.exe",
            "includePath": [
                "${workspaceFolder}"
            ],
            "defines": [],
            "browse": {
                "path": [
                    "${workspaceFolder}"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": ""
            },
            "cStandard": "c11",
            "cppStandard": "c++17"
        }
    ],
    "version": 4
}
```

### 编译代码
要想在vscode里面编译代码并运行，首先需要生成`tasks.json`配置文件。
+ 打开命令行(`Ctrl+Shift+P`)
+ 选择`Tasks: Configure Tasks... `,点击`Create tasks.json file from templates`，出现几种模板，选择`Others`，会在.vscode文件夹内生成`tasks.json`文件
+ 编辑`tasks.json`文件，使其内容如下：
```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Compile",
            "type": "shell",
            "command": "gcc",
            "args": [
                "-g","hello.c",   //指定编译源代码文件                    
                "-o","test.exe", // 指定输出文件名，不加该参数则默认输出a.exe
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
        }
    ]
}
```
+ 然后点击菜单栏中的`任务--运行生成任务(Ctrl+Shift+B)`即可生成对应的exe。
+ 在终端shell里面输入`./test`即可运行并输出结果。

### 代码DEBUG
进入调试窗口 → 点击调试配置按钮 (上方右上角会有红点提示) ，选择 `C++(GDB/LLDB)` (如果使用 MSVC 编译，可以选择`C++(Windows)`) → 此时会生成调试配置文件 `launcher.json` ,如下图所示进行配置,然后再对应的地方添加断点就能进行调试了。
```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/debug.exe", //需debug的目标程序
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "C:/Program Files/mingw-w64/x86_64-8.1.0-release-posix-seh-rt_v6-rev0/mingw64/bin/gdb.exe", //gdb的路径
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            // task.json 中指定的调试目标文件编译命令
            // 这样可以在调试前免去手动执行 build-debug 的一步
            "preLaunchTask": "build-debug"
        }
    ]
}
```

## Markdown环境的配置
Markdown是一种程序员圈内比较流行的书写格式，本博客的文章全部用Markdown格式书写，而vscode本身就自带对Markdown格式的支持，无需任何插件就能同步显示转换后的效果。当然也有很多插件能够提升用户体验，这部分可以参照[官方的帮助site](https://code.visualstudio.com/docs/languages/markdown).

## Python环境的配置
vscode对Python的支持比较全面，基本上官方的插件就能满足一个轻量级项目的大部分的需求。

### 安装Python插件
打开vscode，最左侧sidebar中点击扩展(Extensions)，查找python插件，下载安装排在第一个的插件，并点击reload。

### 安装Python解释器
vscode自身不带有Python的解释器，你可以按照需求安装对应版本的Python解释器，在[官方网站](https://www.python.org/downloads/)就能下载各个不同版本的解释器。

### 编译并运行代码
新建一个Python文件，例如`hello.py`，写一段简单的代码，比如`print("Hello World")`,左下方就能看见目前使用的Python解释器版本，如果系统里面安装了多个Python版本的话可以在此处进行自定义。此时右下角可能弹出`未安装pylint之类的信息`,点击安装即可（安装过程中若提示`UnicodeDecodeError: 'utf-8' codec can't decode byte 0xd7 in position 48: invalid continuation byte`错误，则只需把问题文件问题位置中的`return s.decode('utf_8')` 改为 `return s.decode('gbk')`即可，这是因为windows中文系统的编码问题）。
直接F5就能运行或者打断点调试，非常方便。
