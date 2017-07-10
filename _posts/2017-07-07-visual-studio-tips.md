---
layout: post
title: "Visual Studio使用tips"
tagline: ""
description: "Visual Studio使用tips"
category: VS
tags: [VS,tips,IDE]
last_updated: 2017-07-10
---

Visual Studio在Windows平台上是非常优秀的一款IDE软件，在其使用过程中掌握如下Tips非常有助于快速有效的开发（下列提及的tips均在Visual Studio 2015下通过测试，理论上VS 2013以后的版本应当都支持下列功能）。

# 主题的更换

VS的默认主题为白色主题，想要变更主题，只需顺序点击菜单栏“工具”-“选项”-“环境”-“常规”，将右侧的颜色主题选为深色即可。

# 字体的更换

在“工具”-“选项”-“环境”-“字体和颜色”选项卡中，更改对应的字体和大小选择即可，推荐Consolas字体，12号大小。

# VAssistX的安装

VAssistX是一个非常优秀的插件，提供语法高亮，代码补全等功能。[VAssistX爱心分享版]({{site.url}}/assets/other/Visual Assist X_10.9.2118.zip)同时还能在VS中显示符合linux kernel代码风格的80字符分界线。具体操作如下：菜单栏-VAssistX-Visual Assist Options-Display，选中"Display indicator after column"，并选择相应的列数以及分隔线颜色即可。

# 代码缩进的显示插件

在分支较多的情况，要想快速看懂代码结构，缩进显示是一个非常高效的工具。"[Indent Guides for Visual Studio](http://indentguide.codeplex.com/)"是一个优秀的插件，网站上下载后双击安装即可使用。（介于CodePlex网站将于2017.12.15关闭，此处对[Indent Guides for VS2015]({{site.url}}/assets/other/IndentGuide v15.vsix)做了备份）

