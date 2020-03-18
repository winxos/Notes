# **N次幂级数和函数通向公式推导及编程实现**

> 本文为本人2004年左右大一时对幂级数和函数通向公式的推导，现整理为markdown格式文档，分享给大家。
>
> winxos 2016-12-02

想必大家都知道：

$$\mathbf{1 + 2 + 3 + \ldots +}\mathbf{n}\mathbf{=}\frac{\mathbf{n}\mathbf{(}\mathbf{n}\mathbf{+ 1)}}{\mathbf{2}}$$

大多数人也知道：

$$\mathbf{1}^{\mathbf{2}}\mathbf{+}\mathbf{2}^{\mathbf{2}}\mathbf{+}\mathbf{3}^{\mathbf{2}}\mathbf{+ \ldots +}\mathbf{n}^{\mathbf{2}}\mathbf{=}\frac{\mathbf{1}}{\mathbf{6}}\mathbf{n}\left( \mathbf{1}\mathbf{+}\mathbf{n} \right)\left( \mathbf{1}\mathbf{+}\mathbf{2}\mathbf{n} \right)$$

但是更高次幂级数的话，知道的人就不多了。

我在2004-2005年的时候推导出了任意次幂和函数的递归算法，但由于当时不会编程所以没有编写相关的程序，今天整理电脑时发现了一年多前写的一个求此公式的算法，但当时由于种种原因只实现了系数的数值解，刚好下午无事，便重写了一个分数类和多项式类进行了替换，很开心的发现这个程序工作的非常好^_^

实际上在当时我推导出公式后到图书馆查了一下，发现几百年前伯努利就推导出了任意次幂和函数的公式，还引入了一个伯努利级数，但是当时没有看到更详细的资料。可能我的方法跟其他的人重复了，但是我还是很开心自己解决了一个高中时代的疑惑。

在这里我大致描述一下算法：

**基本原理就是利用二次项展开，然后利用极差原理根据系数和低级别的公式构造出需要的次方的和函数。**

具体描述：比如说我们求

$$\sum_{\mathbf{i = 1}}^{\mathbf{X}}\mathbf{i = ?}$$

根据常识，我们知道一个幂级数的和函数最高次方一定比最后一项高一次，所以我们先**假定**

$$\mathcal{F}_{\mathbf{1\ }}\mathbf{(X) =}\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i =}\mathbf{X}^{\mathbf{2}}}$$

对于n次幂和函数很容易知道有如下的性质：

$$\mathcal{F}_{\mathbf{n}}\left( \mathbf{X} \right)\mathcal{- F}_{\mathbf{n}}\left( \mathbf{X - 1} \right)\mathbf{=}\mathbf{X}^{\mathbf{n}}$$

因此根据我们先前的假定$\mathcal{F}_{1\ }(X) = \sum_{i = 1}^{X}{i = X^{2}}$，我们将$\mathcal{F}_{1\ }\left( X \right) = X^{2}$代入上式得到：

$$\mathcal{F}_{\mathbf{n}}\left( \mathbf{X} \right)\mathcal{- F}_{\mathbf{n}}\left( \mathbf{X - 1} \right)\mathbf{=}\mathbf{X}^{\mathbf{2}}\mathbf{-}\left( \mathbf{X - 1} \right)^{\mathbf{2}}\mathbf{= 2X - 1}$$

发现相差并非我们需要的X，但是我们发现数量级是一样的，我们只需要进行系数上的修改就可以。相当于我们要做一个如下的线性变换：

$$\mathbf{2X - 1}\mathbf{X}$$

**所以我们要对原函数**$\mathcal{F}_{\mathbf{1\ }}\left( \mathbf{X} \right)\mathbf{=}\mathbf{X}^{\mathbf{2}}$**进行如上的规则变换，先加上1再除以2得到X**，***但是我们要注意这里操作的都是和函数，所以说加1指的是加上做差后会产生1的函数，也就是我们的***$\mathcal{F}_{\mathbf{0\ }}\left( \mathbf{X} \right)$***，也就是X，***

因为X-(X-1)==1，这才是产生1的函数，乘法和除法进行运算的时候只用直接全部系数变换就可以。

这样我们就得到了如下的计算过程：

a)  ***假定***$\mathcal{F}_{\mathbf{1\ }}\mathbf{(X) =}\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i =}\mathbf{X}^{\mathbf{2}}}$***，得到极差***$\mathbf{2X - 1}$

b)  ***制定变换规则***$\mathbf{2X - 1}\mathbf{X}$

c)  ***将假定函数进行规则变换***$\frac{\mathbf{F}_{\mathbf{n}}\mathbf{(X) + X}}{\mathbf{2}}\mathbf{\ }\mathcal{F}_{\mathbf{1\ }}\left( \mathbf{X} \right)\mathbf{=}\frac{\mathbf{x}^{\mathbf{2}}}{\mathbf{2}}\mathbf{+}\frac{\mathbf{x}}{\mathbf{2}}$***完成！***

仿造上面的过程，我们再来计算一下

$$\sum_{i = 1}^{X}{i^{2} = ?}$$

1)  我们先假设$\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i}^{\mathbf{2}}\mathbf{=}\mathbf{X}^{\mathbf{3}}}$

2)  得到极差$\mathbf{X}^{\mathbf{3}}\mathbf{-}\left( \mathbf{X - 1} \right)^{\mathbf{3}}\mathbf{= 3}\mathbf{X}^{\mathbf{2}}\mathbf{- 3X + 1}$

3)  得到变换公式$\mathbf{3}\mathbf{X}^{\mathbf{2}}\mathbf{- 3X + 1}\mathbf{X}^{\mathbf{2}}$

4)  我们已经得到$\mathcal{F}_{\mathbf{1\ }}\left( \mathbf{X} \right)\mathbf{=}\frac{\mathbf{x}^{\mathbf{2}}}{\mathbf{2}}\mathbf{+}\frac{\mathbf{x}}{\mathbf{2}}\mathbf{,\ }\mathcal{F}_{\mathbf{0\ }}\left( \mathbf{X} \right)\mathbf{= X}$ ，逆向替换我们得到

$$\mathcal{F}_{\mathbf{2\ }}\left( \mathbf{X} \right)\mathbf{=}\frac{\mathbf{X}^{\mathbf{3}}\mathbf{+ 3}\mathbf{F}_{\mathbf{1\ }}\mathbf{(X) -}\mathbf{F}_{\mathbf{0}}\mathbf{(X)}}{\mathbf{3}}\mathbf{=}\frac{\mathbf{1}}{\mathbf{3}}\mathbf{X}^{\mathbf{3}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{2}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{6}}\mathbf{X}^{\mathbf{1}}\mathbf{\ }$$

**更高次的通过以上的方法也可以进行构造出来了。**

winxos 2010-2-1



**附：10次以内的和函数。**

$$\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i}^{\mathbf{2}}\mathbf{=}}\frac{\mathbf{1}}{\mathbf{3}}\mathbf{X}^{\mathbf{3}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{2}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{6}}\mathbf{X}^{\mathbf{1}}\mathbf{\ }$$

$$\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i}^{\mathbf{3}}\mathbf{=}\frac{\mathbf{1}}{\mathbf{4}}\mathbf{X}^{\mathbf{4}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{3}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{4}}\mathbf{X}^{\mathbf{2}}}$$

$$\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i}^{\mathbf{4}}\mathbf{=}\frac{\mathbf{1}}{\mathbf{5}}\mathbf{X}^{\mathbf{5}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{4}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{3}}\mathbf{X}^{\mathbf{3}}\mathbf{-}\frac{\mathbf{1}}{\mathbf{30}}\mathbf{X}^{\mathbf{1}}}$$

$$\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i}^{\mathbf{5}}\mathbf{=}\frac{\mathbf{1}}{\mathbf{6}}\mathbf{X}^{\mathbf{6}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{5}}\mathbf{+}\frac{\mathbf{5}}{\mathbf{12}}\mathbf{X}^{\mathbf{4}}\mathbf{-}\frac{\mathbf{1}}{\mathbf{12}}\mathbf{X}^{\mathbf{2}}}$$

$$\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i}^{\mathbf{6}}\mathbf{=}\frac{\mathbf{1}}{\mathbf{7}}\mathbf{X}^{\mathbf{7}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{6}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{5}}\mathbf{-}\frac{\mathbf{1}}{\mathbf{6}}\mathbf{X}^{\mathbf{3}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{42}}\mathbf{X}^{\mathbf{1}}}$$

$$\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i}^{\mathbf{7}}\mathbf{=}\frac{\mathbf{1}}{\mathbf{8}}\mathbf{X}^{\mathbf{8}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{7}}\mathbf{+}\frac{\mathbf{7}}{\mathbf{12}}\mathbf{X}^{\mathbf{6}}\mathbf{-}\frac{\mathbf{7}}{\mathbf{24}}\mathbf{X}^{\mathbf{4}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{12}}\mathbf{X}^{\mathbf{2}}}$$

$$\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i}^{\mathbf{8}}\mathbf{=}\frac{\mathbf{1}}{\mathbf{9}}\mathbf{X}^{\mathbf{9}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{8}}\mathbf{+}\frac{\mathbf{2}}{\mathbf{3}}\mathbf{X}^{\mathbf{7}}\mathbf{-}\frac{\mathbf{7}}{\mathbf{1}\mathbf{5}}\mathbf{X}^{\mathbf{5}}\mathbf{+}\frac{\mathbf{2}}{\mathbf{9}}\mathbf{X}^{\mathbf{3}}\mathbf{-}\frac{\mathbf{1}}{\mathbf{30}}\mathbf{X}^{\mathbf{1}}}$$

$$\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i}^{\mathbf{9}}\mathbf{=}\frac{\mathbf{1}}{\mathbf{10}}\mathbf{X}^{\mathbf{10}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{9}}\mathbf{+}\frac{\mathbf{3}}{\mathbf{4}}\mathbf{X}^{\mathbf{8}}\mathbf{-}\frac{\mathbf{7}}{\mathbf{10}}\mathbf{X}^{\mathbf{6}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{4}}\mathbf{-}\frac{\mathbf{3}}{\mathbf{20}}\mathbf{X}^{\mathbf{2}}}$$

$$\sum_{\mathbf{i = 1}}^{\mathbf{X}}{\mathbf{i}^{\mathbf{9}}\mathbf{=}\frac{\mathbf{1}}{\mathbf{11}}\mathbf{X}^{\mathbf{11}}\mathbf{+}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{10}}\mathbf{+}\frac{\mathbf{5}}{\mathbf{6}}\mathbf{X}^{\mathbf{9}}\mathbf{-}\frac{\mathbf{1}}{\mathbf{1}}\mathbf{X}^{\mathbf{7}}\mathbf{+ \ }\mathbf{X}^{\mathbf{5}}\mathbf{-}\frac{\mathbf{1}}{\mathbf{2}}\mathbf{X}^{\mathbf{3}}\mathbf{+}\frac{\mathbf{5}}{\mathbf{66}}\mathbf{X}^{\mathbf{1}}}$$

### 附：python代码

``` python
#coding:utf-8
'''
幂级数和函数python求解
利用递归进行实现
winxos2012-10-9
'''
import numpy as np
from fractions import Fraction as F
tb={0:np.poly1d([F(1),F(0)])}
def binomialCoeff(n, k): #计算二项式展开
   result = 1
   for i in range(1, k+1):
       result = result * (n-i+1) / i
   return result
def getDiff(n): #得到高次级差
   return map(lambda i:-binomialCoeff(n,i)*(-1)**i,range(1,n+1))
def powsum(n):
   if tb.has_key(n):return tb[n] #动态规划
   g=getDiff(n+1)
   an=np.poly1d([F(1)]+[0]*(n+1)) #初始构造
   for i in range(1,len(g)):
       an-=powsum(len(g)-1-i)*g[i]
   tb[n]=an/g[0]
   return tb[n]
if __name__ == "__main__":
   for i in range(10):
       g=powsum(i)
       print "sum x^%d="%i,g.coeffs
       print g
   print "sum x^30=",powsum(30).coeffs
```

