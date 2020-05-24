---  
layout: post  
title: np.random.seed()
color: 888888
description: 随机数产生的算法与系统有关，在 Windows、macOS 或者 Linux 上即便是随机种子一样，产生的随机数也不一样。
---  
  
写作业时用到了 numpy 的 np.random.seed()  
网上一搜发现解释并不清晰，还有人提出了一些质疑，于是自己测试了一番。我的理解是：  
产生的随机数是伪随机数，从设置种子 seed 的那一刻起，每批生成的随机数都有某些固定规律，虽然人脑很难猜到，但是计算机可以复现。比如：  
![1](/images/nprandomseed-1.png)  
![2](/images/nprandomseed-2.png)  
说明从设置了 seed 以后，无论一批有多少个数字，都可复现；只要知道第几批生成，所有数字也都可以复现。  
另外，随机数产生的算法与系统有关，在 Windows、macOS 或者 Linux 上即便是随机种子一样，产生的随机数也不一样。而 Ng 老师的作业设置的“Expected Output”看起来是从 Windows 操作系统上得到的。  
