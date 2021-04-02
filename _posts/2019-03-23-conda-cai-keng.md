---
layout: post
title: Conda 踩坑
color: 4CAF50
description: 默认源速度有点慢，可以换成清华的。 
---

默认源速度有点慢，可以换成清华的。  

``` cmdline
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/  
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/  
conda config --set show_channel_urls yes  
pip install pip -U  
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple  
```

<del>
**嗨呀，清华说没能得到授权，不再提供了。**
</del>  
**最新消息：清华取得了镜像授权，重新开张。**  
**最新消息：清华的镜像经常断掉，可以考虑 [opentuna](https://opentuna.cn/) [北外](https://mirrors.bfsu.edu.cn/) 都是姊妹**

默认不激活 base  
`conda config --set auto_activate_base false`
Linux 和 macOS 安装时注意选择路径，由于没有卸载功能，换目录的时候会遭遇一系列的问题。  

macOS 下有一个比较坑的地方就是，默认用 conda install 安装的 TensorFlow spicy numpy 等都是支持 mkl 的版本，mkl 是 Intel 搞的可以加速科学计算的东西，但是好心办坏事，在完成某些作业时会发生如下错误：  

```
OMP: Error #15: Initializing libiomp5.dylib, but found libiomp5.dylib already initialized.  
OMP: Hint: This means that multiple copies of the OpenMP runtime have been linked into the program.  
```

解决的办法呢，比较尴尬，其一是忽略这项错误，但是会造成没有任何提示地发生计算错误  
`os.environ['KMP_DUPLICATE_LIB_OK']='True'`  
或者呢 conda uninstall mkl 然后用 pip 重新安装 TensorFlow matplotlib 等你需要的库。  
  
Windows 下就不怎么好玩。安装的最后一步有两个选项，第一个是问你是否加入环境变量，默认是不勾选，你要执意选上的话，底下的提示还会变红警告你。问题也就处在这，如果不勾选，系统的 Terminal 也就是 Power Shell 找不到 Conda，重新安装的话还提示目录不为空，让换路径，只好卸载重来。之后打开 Visual Studio Code，Power Shell 虽然知道 Conda 是啥了，但是依然报错。而用 cmd 就没有问题，所以需要在 VS Code 的配置文件中加入一行  
`"terminal.integrated.shell.windows": "C:\\Windows\\System32\\cmd.exe"`  
当然，不要忘了给上一行配置末尾加上逗号。  

使用 conda 安装 tensorflow 的方便之处是，他会自动安装 gpu 版本需要的那几库，也会给 cpu 版本自动装上 MKL 库，有人测试说 tensorflow 能快 8 倍，此外也能让 numpy scipy 那些库加速。 

在 Ubuntu 上安装基本上比较顺，GPU 的驱动和 cuda 等依赖库还是需要按照 TensorFlow 官网的方法安装（这样最简便）。  
在 Ubuntu 上安装 Conda 不建议 sudo 安装，否则像我卡在了莫名其妙的地方，安装完了以后 Conda 需要管理员运行，成为管理员呢，又找不到 Conda 命令。只好 rm -rf 移除 Conda 安装目录，重新用普通用户权限安装。  

2019-08-09 补充  
目前 conda 还不支持 tensorflow-gpu 2.0 beta， 只能通过 pip 安装。可是这样就没法自动安装 cudnn 和 cudatoolkit。后两个库还是可以通过 conda 安装，可是版本又会出现混乱，tensorflow-gpu 只能支持到 cuda-10.0，而最新的 cuda 是 10.1。经过一番尝试，此刻可行的办法如下：（顺序不能变，第三行的命令会降级第二行安装好的版本，只有这样才能成功。）  

```
pip install tensorflow-gpu==2.0.0-beta1  
conda install cudnn=7.6.0  
conda install cudatoolkit=10.0.130  
```

可能过一阵子再更新，版本号要求的又不同了；又或者正式版释出就不需要下折腾了。  
此外，判断 TensorFlow 能不能用 GPU 可以通过下面的代码：  
`tf.test.gpu_device_name()`  
输出 `'/device:GPU:0'` 就对了。  

最后还是建议大家少折腾环境，多学习知识，多动手实操。听说 TensorFlow 在 Ubuntu 上跑得更有效率，但是呢，很可能实际卡在糟糕的结构设计，而系统之间的差别相比之下微不足道的。而且本地那一两块显卡可能很快就变成瓶颈了，到时候还得转云环境（如 Colab）。  
