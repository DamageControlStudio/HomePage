---
layout: post
title: 两个双系统折腾一整天
author: 扑兔
color: 00897B
description: 集齐 Win 10 ＋ Ubuntu + macOS Mojave ＋ Win 7 是个什么鬼
---

今天“捉妖”，把 Alienware 整成了 Win 10 ＋ Ubuntu 双系统，为了玩游戏和炼丹，  
然后又把 Mac mini 整成了 macOS Mojave ＋ Win 7 双系统，为了学习和玩红警，  
以及兼容至今仍神一样存在要求 IE6 的 OA 系统。下面记录一下这一折腾就是一整天遇到的几个坑：  

Alienware 是 SSD ＋ 指针硬盘，启动项里硬盘模式不能选择 Raid On，否则装系统时看不到 SSD。  
先进 PE 系统把硬盘分好区（先备份 Win 10 的激活信息），然后装 Windows 10，然后装 Ubuntu。  
后者在 Alienware R5 - R7 上是不能正常关机的，发现是 dw_i2c 引起的，然后上百度必应一顿搜索，全是他娘的不相关的解决方案。  
最后靠浙江大学搭建的镜像站登上了 Google，第一条就是解决方案：```sudo nano /etc/default/grub```  
把其中的一行改为：```GRUB_CMDLINE_LINUX_DEFAULT=“quiet initcall_blacklist=dw_i2c_init_driver”```  
然后更新 grub，用 ```sudo update-grub```  
更新：原版的 Dock 和 Top Bar 都用插件隐藏起来的话图标会重叠，或者休眠后出现两套 Dock。  
解决办法就是卸载 ```sudo apt remove gnome-shell-extension-ubuntu-dock```  然后用默认的居中 Dock。  

Mac mini 是 2014 款的，我先 command ＋ R 启动到恢复模式，把硬盘格式化，然后重启，  
为了让系统不能恢复 Mojave。然后再开机，进入地球联网恢复模式，  
连接 Wi-Fi，这样系统能恢复成 Yosemite 10.10.5，怎么样，很古老了吧？这样的好处是自带 Boot Camp 5 的版本，能装 Win 7。  
不出意外的话至少两小时，进入 Yosemite，用 Boot Camp 制作 Win 7 的安装 U 盘，  
U 盘慢的话这个过程也很久，我等了一个小时。  
制作好之后先别着急安装！把 $WinPEDriver$ 和 BootCamp 这两个文件夹删掉，  
因为目前苹果会自动安装6.0版本的驱动，会导致著名的 applessd.sys 错误，或者进入 Win 7 安装界面时键盘鼠标无反应。  
上苹果官网下载 bootcamp5.1.5769.zip，大约 543 Mb，解压出来的 $WinPEDriver$ 和 BootCamp 丢到 U 盘里面去，  
这回就可以放心安装 Win 7 了。  
这个即将被官方放弃支持的系统咱就别瞎搞了，当作电子收藏用，  
因为之后 macOS 再更到 Mojave，你就再也不能 Boot Camp 安装 Win 7 了，目前只支持 Win 10 了哟。  
要不然倔强如我，折腾一天如上步骤寻开心。  
