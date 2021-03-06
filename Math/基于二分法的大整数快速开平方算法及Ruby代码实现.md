# **基于二分法的大整数快速开平方算法及Ruby代码实现**

> 本文为本人2009年时对快速开方算法进行分析得到的文章，现整理成markdown格式分享给大家。
>
> winxos 2016-12-02

## 摘要：

本文提出了一种利用二分法实现的非迭代大整数快速开平方算法，并给出了相应的**Ruby**语言代码实现。

**关键词：大整数；平方根；非迭代**

## 1 算法基本原理

假设数$n=ab$, a,b分别为数n的前半段和后半段，若数$b$的位数为$k$，

则n可以表示为

$n=a\times10^k+b$ 											①

我们可以得到

$n^2=(a\times10^k+b)^2=a^2\times10^{2k}+2\times10^k\times a\times b+b^2$ 		②

我们将式②分成两部分

**Ⅰ**. 	$a^2\times10^{2k}$

**Ⅱ**. 	$2\times10^k\times a\times b+b^2$

对于式 Ⅰ 令$a=a+1$，式 Ⅰ 变成

**Ⅲ**. 	$ (a+1)^2\times10^{2k}=a^2\times10^2k+2\times10^{2k}\times a+10^{2n}$

对于式 **Ⅲ** ，因为$b<10^k$，所以$2\times10^{2k}\times a+10^{2n} > $ Ⅱ

所以式 **Ⅲ** > Ⅰ+Ⅱ，由此我们可以知道数 $n^2$ 以后的结果$>10^{2k}$ 的数字是由 $a$ 唯一确定，对于已知$n^2$，求n时，我们就可以先将$n^2$按 **Ⅰ** 和 **Ⅱ** 的分法分成两段，通过分好后的Ⅰ我们可以求出$a$，并求得 Ⅰ 的余数 $r$，然后根据

**Ⅱ** $= n^2-a^2\times 10^{2k}=r\times10^{2k}+n^2\mod10^{2k}=2\times10^k\times a\times b+b^2$

$a$ 又由 **Ⅰ** 求出，我们可以求得 $b$，从而求得 $n$。

由于 $k$ 是任意的，则我们可以将数 $n^2$ 分成很多段，逐步的求出 $a_n$ 以及 $b_n$。实

际上我们可以先将 $k$ 设置的比较大，那样式 **Ⅰ** 就会比较小，可以直接简单的小数字开方得到$a_n$，我们将 $k$ 减小，这样取得的式 **Ⅰ** 就会比之前取得的更大，这时我们求 $a_m$ 的时候就可以把式 **Ⅰ**当成 $m^2$ 来处理，通过我们之前求得 $k$ 较大时候的 $a$ 以及式 **Ⅱ**，可以求出 $m^2$ 开方的后半部分 $b_m$ ，然后我们继续减小 $k$，让 $a_i$ 等于上一次式 **Ⅰ** 开方得到 $m$，直到将 $n$ 全部求出。

## 2 基本开平方算法过程演示

例如 $n=12345678$求$\sqrt{n}$

将 $n$ 分节成 **12 34 56 78**，便于观察。

1. 取 $k=3$，式 **Ⅰ** =12，很容易求得 $a_1=3$ ，余数 $r_1=3$

2. 取 $k=2$，式**Ⅰ** =1234，要求 $a_2$，我们将式 **Ⅰ** 当成 $m^2$，求得 $m^2$ 展开的式 **Ⅰ**$_m =12$由于**Ⅰ**$_m=12$ 对应的 $a_m$ 我们在第一步中已经求出，$a_m=3$ ，我们得到:

   **Ⅱ**$_m=3\times10^2 + 34 = 334 = 2\times10 \times3\times b + b^2=60\times b + b^2$

   我们求得$b=5$，这样我们就求得了$a2=35$，余数$r_2= 334-60\times5-5\times5 = 9$

3. 取 $k=1$，式**Ⅰ **$=123456$，要求$a_3$，将式**Ⅰ**当成$m^2$，取$m^2$展开的式Ⅰ$_m=1234$，由于

   Ⅰ$_m=1234$对应的$a_m$我们在第二步中已经求出，$a_m=35$，我们得到

   Ⅱ$_m=9\times10^2 + 56 = 956 = 2\times10\times35\times b + b^2 = 700\times b + b^2$

   求得$b=1$，这样我们就求得$a_3=315$，余数$r_3= 956 – 700 – 1 = 255$


4. 取$k=0$，式Ⅰ$=12345678$，要求$a_4$，将式 **Ⅰ** 当成$m^2$，取$m^2$展开的式

   Ⅰ$_m=123456$，第三步中已经求出Ⅰ$_m=123456$对应的$a_m=351$，$r_3=255$我们得到

   Ⅱ$_m=255\times10^2+78=25578=2\times10\times351\times b + b^2=7020\times b+b^2$

   求得$b=3$，这样我们就求得$a_4=3513$，余数$r_4~= 25578 - 7020\times3 – 9 = 4509$

至此，我们就求得了$\sqrt{12345678}=3513$ ，余数 $r=4509$。

## 3 基本开平方算法效率分析

​	假定待开平方数 $n$，数$n$的数位长度为Len，该算法需要将数$n$分成$\left\lceil \text{Len}/2 \right\rceil$段，每段需要计算常数项乘法及加减法，$\mathrm{\theta}\mathrm{(n) = log(n)}$，由于在计算$a$的时候涉及了大数运算，还要加上大数的计算复杂度。

​	在实际应用中已经有许多现成的大数库可供调用，如C/C++可以调用**apfloat ，gmp**等比较成熟的算法库，或者可以直接使用支持大数运算的语言进行编程，如**Ruby ,Python**，本文后面的算法就是基于**Ruby**语言实现的。

#### C++实现代码：

``` c++
int SmallSqrt(int n) //快速100以内开方
{
  int ret;
  ret=(n>=1)+(n>=4)+(n>=9)+(n>=16)+(n>=25)+(n>=36)+(n>=49)+(n>=64)+(n>=81);
  return ret;
}
/*
快速开方函数，原理为ab*ab=a*a*100+2*a*10*b+b*b=a*a*100+20*a*b+b*b,注意其中的各部分的大小
比如说求123456开方
①分成三段12 34 56，先对12开方等到3，可以确定百位a=3，余数=12-3*3=3
②求第二段加上余数得到3 34，余数是由20*a*b+b*b决定，而所以334>(20*3*b+b*b),求得b=5，余数=334-20*30*5-5*5=9
③让a=a*10+b，即a=35，余数为956，重复前面的过程，得到b~956/700=1，
④得到123456的开方整数部分为351
当然如果想算小数也可以继续算下去，而且待开方的数字理论上可以很长，比如>100位，算法一样。
算法为今天晚上上信号测试课时想到。
winxos 2009-03-31
*/
long FastSqrt(long n) //快速整数开方程序，原理如上
{
  long n2,temp;
  int head,a,b,remain=0,len=0;
  int i,j;
  n2=n;
  while (n2/100)//计算分节长度
  {
     n2/=100;
     len++;
  }
  head=n2;
  a=SmallSqrt(head);//对首节开方
  remain=head-a*a;
  for (i=len-1;i>=0;i--)
  {
     temp=1;
     for (j=0;j<i;j++)//
     {
      temp*=10;
     }
     head=(n/(temp*temp))%100+remain*100;
     b=head/(20*a);//20ab要大于b*b很多
     while ((20*a*b+b*b)>head)//20ab+b*b要小于余数
     {
      b--;//如果b*b影响了b的数值，要减一
     }
     remain=head-20*a*b-b*b;
     a=a*10+b;
  }
  return a;
}
```



## 4 基于二分法的改进加速开平方算法原理

​	在我们使用上面提到的基本开平方算法时，我们发现经过一次迭代只能计算出一位长度的$b$，而实际上根据我们的$a$，可以一次计算出多位的$b$，特别是当$a$越大时，一次计算出的$b$位数就可以越多，类似于二分法，可以大量的减少迭代次数，极大的提高的算法的效率。

对于式Ⅱ$=2\times10^k\times a\times b+b^2$在计算的过程中余数$r$和前段$a$已知的情况下，解$b$需要解一个一元二次方程，在基本算法中由于$b$每次都是一位数，而$a$会越来越大，所以

$2\times10^k\times a \times b \gg b^2$ （注：$\gg$ 表示远远大于）

所以我们可以直接的通过$2\times10^k\times a\times b$ 就能方便的大致计算出$b$，然后再进行校验，就省去了解一元二次方程的过程。

采用加速算法后，随着$b$的增大，$2\times 10^k\times a\times b$与$b^2$的差距会变小，如果$b$取的太大，$b^2$的影响较大时，就增加了我们计算b的难度，所以我们选取$b$与$a$等长，这样$b^2$的影响就不至于太大，我们仍然可以用$2\times10^k\times a\times b$计算出$b$的初值，然后进行校验得到准确的$b$值。

在实现过程中要注意几点：

1. 如果$n$的长度Len不为$2^m$的形式，根据Len的奇偶性，我们应该在$n$末尾添加一定数目的0，使其长度Len达到$2^m$或$2^m-1$（奇数长度时为$2^m-1$），便于后面的分段计算能够顺利进行。待计算完后再去除一定长度的0进行还原（不还原实际上是获得了额外的精度以及乘了一个$10^K$）。
2. 分段长度初始取2（如果Len为奇数，则n前面要补一个0），之后每次分段长度为上次的2倍，意思就是说10次迭代我们就可以计算$2^{10}$长度的数的开平方。
3. 计算$b$的时候先根据$2\times10^k\times a\times b$估计出b的上限，然后依次让$b=b-1$校验出准确的$b$的值

## 5 基于二分法的改进加速开平方算法过程演示

例如$n=12345678$，求$\sqrt{n}$

1. 取$m^2=12$，解得$a_1=3，r_1=3$
2. 取$m^2=1234$，由于$a_1=3$，解得$b_2=5$，得到$a_2=35，r_2=255$
3. 取$m_2=1234 5678$，由于$a_2=35$，解得$b_3=13，r_3=4509$

这样我们就求得$\sqrt{12345678}=3515^2$，余数$r=4509$

## 6 基于二分法的改进加速开平方算法效率分析

​	假定待开平方数$n$，数$n$的数位长度为Len，该算法需要将数$n$分成log(Len/2)段，每段需要计算常数项乘法及加减法，不考虑大数运算复杂度，时间复杂度$\mathrm{\theta}\mathrm{(n) = log(log(n))}$

## 7 基于二分法的改进加速开平方算法的Ruby语言实现

```python
#a fast sqrt program use ruby
#by winxos 2009-04-02
st=987654321**666
n=String(st*st)
len=n.length
max=((Math.log(len)/Math.log(2))-0.05).floor #get loop times
max2=2**(max+1)-len
max2.times(){n=n+"0"} #change n's length to 2**k
coef=2-len%2 #odd ,even use two methods
print("calc (987654321^666)^2\n")
print("len:",len," max:",max,"\n")
print("source:     ",st,"\nsource^2: ",n,"\n")
start=Time.now  #timer
a=Math.sqrt(n[0,coef].to_i).floor
remain=n[0,coef].to_i-a*a
coef%=2
1.upto(max){|i|
  steps=10**(2**(i-1))
  remain=remain*steps*steps+n[2**i-coef,2**i].to_i
  b=remain/(2*steps*a)  #first b
  b=b-1 while (remain-2*steps*a*b)<b*b #check b
  remain=remain-2*steps*a*b-b*b #get remain to continue calc
  a=a*steps+b
}
elapsed=Time.now-start  #the end time
result=String(a)
result=result[0,result.length-max2/2].to_i  
#restore result,trim end's zero
print("ans :        ",result,"\nans^2:     ",result*result,"\n")
print("used:",elapsed," s.\n")
```

