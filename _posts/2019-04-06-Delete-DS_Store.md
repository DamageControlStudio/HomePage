---
layout: post
title: 干掉烦人的 .DS_Store
---

托管代码到 GitHub 的时候，.DS_Store 这个文件老是出来捣乱。倒是可以忽略，但是 .gitignore 管不着其他不同目录下的 .DS_Store 文件。  
解决办法就是建立 ~/.gitignore_global ，内容如下：  
```
# .gitignore_global

.DS_Store
.DS_Store?
*.swp
._*
.Spotlight-V100
.Trashes
```  
然后在 ~/.gitconfig 中添加 [core] 内容如下：  
```
[user]
	name = xx yy
	email = xy@abc.de
[core]
	excludesfile = ~/.gitignore_global
```  
duang ，世界安静了。