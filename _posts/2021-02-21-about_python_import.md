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

最终的效果是不论 2.x 还是 3.x 中的文件，你都应该只选择一种使用模式，即包模式（使用相对导入）或程序模式（使用简单导入），并将真正的包模块文件单独放到一个子目录中，与顶层脚本文件隔离开来。  
通过绝对导入使用完整的包路径，并且假定包的根目录位于模块搜索路径上，来替代包相对导入语法或简单导入：  

``` Python
# works in both program and package mode
from system.section.mypkg import mod
```

对于所有的方案，最后一种（完整包路径导入）应该是最具可移植性的也是最强大的。  

在 Python 3.x 中，`from mypkg import spam` 是绝对导入：mypkg 的搜索跳过包路径并且 mypkg 位于 sys.path 中的一个绝对目录中。另一方面，`from . import spam` 是相对导入：spam 的查找只会是相对于该语句所在的包。  

--以上内容摘自《Learning Python 5th》
