---
layout: post
title: "VMware安装Ubuntu16.04系统指南"
tagline: ""
description: "VMware安装Ubuntu16.04系统指南"
category: ubuntu
tags: [vmware,ubuntu]
last_updated: 2017-03-30
---

## 系统安装步骤(没有提及部分的使用建议或默认设置即可)

+ 准备好Ubuntu系统的镜像文件后，打开VMware(测试版本：12Pro，理论其他版本同样适用)，选择新建虚拟机->自定义

![安装步骤1]({{site.url}}/assets/images/20170330-1.png)

+ 到达下图窗口，选择“稍后安装操作系统”

![安装步骤2]({{site.url}}/assets/images/20170330-2.png)

+ 正确选择待安装系统的类型

![安装步骤3]({{site.url}}/assets/images/20170330-3.png)

+ 选择虚拟机安装位置

![安装步骤4]({{site.url}}/assets/images/20170330-4.png)

+ 选择创建新的虚拟磁盘，单个文件，文件位置一般与虚拟机安装位置相同

![安装步骤5]({{site.url}}/assets/images/20170330-5.png)

+ 完成后在虚拟机设置中将CD/DVD项改为使用ISO镜像文件，并添加事先准备好的ISO文件

![安装步骤6]({{site.url}}/assets/images/20170330-6.png)

+ 启动虚拟机，正常全新安装即可(语言推荐选择英文，安装过程中若选择安装更新和第三方工具会延长安装时间)

## 设置虚拟机的共享文件夹

+ 在虚拟机关闭的情况下打开虚拟机设置，添加共享文件夹

![共享文件夹1]({{site.url}}/assets/images/20170330-7.png)

+ 要实现共享文件夹在虚拟机里面的访问，需安装VMware tools，推荐安装open-vm-tools,该工具已在github上开源，Ubuntu命令如下

```
cd ~
apt install open-vm-tools
apt install git gcc make linux-headers-$(uname -r)
git clone https://github.com/rasa/vmware-tools-patches.git
cd vmware-tools-patches
./patched-open-vm-tools.sh
```

+ 安装完成后reboot即可
