---
layout: post
title: np.fromfunction
color: 888888
description: 一句话说清楚 numpy.fromfunction() 函数咋回事
---

```python
def my_function(z, y, x):
    return x * y + z
```

np.fromfunction(my_function, (3, 2, 10))  
第一个参数：my_function  
第二个参数：(3, 2, 10)  
返回值是一个矩阵（假定为 r ），r 的 shape 等于第二个参数--也就是 (3, 2, 10)，r[i][j][k]=my_function(i, j, k)。  
~~对不起，没能仅用一句话讲清~~  
