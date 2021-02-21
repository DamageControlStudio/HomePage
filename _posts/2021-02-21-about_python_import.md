---
layout: post
title: Python 中很容易忽略的关于 import 的重要信息
color: 888888
description: 读书笔记
---

一些文件在 3.x 中不能像 2.x 中那样同时扮演脚本和包模块的角色。  

``` Python
# 下面这行在 2.x 和 3.x 中都只能用于包模式
from . import mod
# 下面这行在 3.x 中只能用于程序模式，也就是顶层脚本才能通过 import mod 的语法来导入位于同一目录下的其他模块
import mod
```

以上内容摘取自《Learning Python 5th》
