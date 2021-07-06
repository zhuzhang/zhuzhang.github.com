---
layout: post
title: 关于浮点数的那些事
---

## __0__ 一些问题
* __0.1__ 计算机是如何表示小数的？
* __0.2__ C语言中，同占用32bit，为什么float的表示范围远大于int？
* __0.3__ 常用的语言中，比如C能得到0.1的精准表示么？
* __0.4__ 计算机中常说的单精度浮点数、双精度浮点数具体是指什么？
* __0.5__ 单精度浮点数能表示的最小正数是多少？
* __0.6__ 计算机中浮点数运算有哪些坑？为什么？如何避免？

***
## __1__ 浮点数的定义
### __1.1__ 数学定义
简单来说，浮**点**(小数点)数是实数的一种表示方式，其形式和科学记数法比较类似，可以理解为浮点数是科学记数法更通用的表达形式，由固定长度的有效数字乘以指数幂组成，表示为：

{% raw %}$${significand}\times{base}^{exponent}$${% endraw %}

其中有效数字`significand`、幂底数`base`、幂指数`exponent`均为整数, 特别地，{% raw %}$${base}\geq{2}$${% endraw %}。

常见的科学记数法幂底数为10，有效数字和幂指数为10进制数字。

实数包含无理数(无限不循环小数)、有理数。一般在使用中，都会在精度和存储空间之间做出取舍，比如"精确到小数点后2位"之类的描述。

在浮点数中，约定有效数字的长度表示为该浮点数的精度，并且定义归一化的浮点数(normalized floating-point numbers)中的有效数字在1到base之间，如base为10的归一化浮点数{% raw %}$$1.528535407\times10^{5}$${% endraw %}，它的精度为10，所以有效数字又可表示为

{% raw %}$$\frac{s}{b^{p-1}}$${% endraw %}

其中s为忽略小数点的整数，p为该浮点数的精度，b是幂底数。

### __1.2__ 实数的其他表示
#### __1.2.1__ 定点数(Fixed-point)
定点数是小数点的位置是固定的，可以理解为浮点数中的exponent是一个固定的整数。一般存储为整数，然后在乘以固定的缩放因子（scaling factor）；

与浮点数的对比：浮点数的小数点位置可以通过exponent来浮动控制。浮点数表示的范围比较大，精度则随着小数点的位置动态调整，而定点数表示的范围和精度都是固定的。定点数的一个优势是，数学运算很方便，实质就是整数的数学运算。

应用场景：因为定点数的局限性，实际应用场景不是特别广泛，适用于某些特定的场景中，比如：
* 不支持浮点计算的处理器，可以用定点数来表示小数
* 商业系统中，货币的表示

#### __1.2.2__ 对数(Logarithmic number systems)
将数表示为其对数形式，用于某些特定的场景，如数字信号处理系统（DSP）。

#### __1.2.3__ 可变浮点数(Tapered floating-point)
一般浮点数中的significand和exponent是用固定长度的位数表示，而对于可变浮点数，这些位数也是可以调整的，虽然灵活性加强了，但是系统的复杂度增加了，几乎无实际应用场景。

除了上面的表示方式，还有很多其他表示方式，一般都是特殊应用场景下的衍生出来的，比如mathematica中的代数系统，按照数学定义表示数，因此可以得到任意精度(根据数学定义演算)的实数表示，前提是存储空间足够。

***
## __2__ 浮点数的表示范围
根据定义，浮点数都是有理数，并且只能精确表示有理数的一个子集。所以对于无理数只能近似表示，比如{% raw %}$$\pi$${% endraw %}，并且对于不在其表示范围内的有理数也只能近似表示，到底能表示那个子集，取决于base的值，比如{% raw %}$$\frac{1}{10}$${% endraw %}能用base为10的浮点数准确表示，但却不能用base为2的浮点数准确表示。

### __2.1__ 归一化浮点数表示范围
对于任意一个归一化浮点数系统(`b`, `p`, `l`, `u`)，其中`b`表示幂底数，`p`表示有效数字的精度，`l`表示幂指数的最小值，`u`表示幂指数的最大值，根据归一化浮点数的定义：

{% raw %}$${significand}=\sum\limits_{i=0}^{p-1}{x_i}{b^{-i}}$${% endraw %}

其中{% raw %}$${x_0}\in[1, b-1],{x_i}\in{[0, b-1]}$${% endraw %}，故 ：

{% raw %}$${significand}\in[1, \frac{b^p-1}{b^{p-1}}]$${% endraw %}

所以对于上述无符号归一化浮点数系统F，有：

{% raw %}$$f\in[b^l,(1-b^{-p})b^{u+1}]$${% endraw %}

ps: IEEE 754协议中一般规定{% raw %}$$l=1-u$${% endraw %}

### __2.2__ 归一化浮点数的分布
直观来讲，对于浮点数的指数幂部分，其值会随着exponent增加呈指数级变化，比如对于base为10的浮点数系统，其值从0.001、0.01、0.1、1、10、100、……呈现指数级变化。

而对于固定精度的有效数字，其划分是固定的，比如base为10，精度为10的浮点数，其最小划分粒度就是{% raw %}$$10^{-9}$${% endraw %}。

所以综合来看，最小划分粒度(相邻两数的间隔)在相同exponent中不变，而随着exponent数值的增大，会呈指数增加，如下图(IEEE 754双精度浮点数)所示：

![floating points](/assets/floatingpoints.png)

***
## __3__ 计算机中浮点数
### __3.1__ 历史
最早包含浮点数运算的系统是1914年的一个用于分析数据的电机系统中，随着计算机的发展，迁移到计算机系统中，并逐步完善。

最初大家的标准都不统一，在计算机中的表示也是各有不同，存储占用的大小也不一样，有36bits的单精度，有72bits的双精度。

在1970s的时候，由于不统一导致的各种兼容性问题，比如不同字数大小、不同表示方式、不同精度等问题让大家意识到标准化这个事情非做不可。

在1980s的时候，基于intel在i8087中数学运算的协处理器单元中的浮点数系统设计，以及同期motorola的68000微处理器中的相关设计，推出了浮点数系统的标准协议IEEE 754，该协议的架构设计师也因此获得1989年图灵奖。

### __3.2__ IEEE 754
#### __3.2.1__ 单精度
![ieee single](/assets/ieee_single.png)

* base为2的归一化浮点数：{% raw %}$${x}=(-1)^s1.f\times2^m$${% endraw %}
* 1bit的符号位s，s=0时为正数，s=1时为负数
* 8bit的幂指数c，其中c=m+127，保留c=255和c=0，用作特殊用途，故l=-126，m=127
* 23bit的小数部分f
* 最小精度{% raw %}$$\epsilon=2^{-23}\approx1.2\times10^{-7}$${% endraw %}
* 最小归一化正浮点数{% raw %}$$UFL=2^l=2^{-126}\approx1.2\times10^{-38}$${% endraw %}
* 最大归一化正浮点数{% raw %}$$OFL=(1-2^{-p})2^{u+1}=(1-2^{-24})2^{128}\approx3.4\times10^{38}$${% endraw %}

值得注意的是，为了避免存储符号位，exponent实际存储为exponent+127

#### __3.2.2__ 双精度
![ieee_double](/assets/ieee_double.png)

* base为2的归一化浮点数：{% raw %}$${x}=(-1)^s1.f\times2^m$${% endraw %}
* 1bit的符号位s，s=0时为正数，s=1时为负数
* 11bit的幂指数c，其中c=m+1023，保留c=2047和c=0，用作特殊用途，故l=-1022，m=1023
* 52bit的小数部分f
* 最小精度{% raw %}$$\epsilon=2^{-52}\approx2.2\times10^{-16}$${% endraw %}
* 最小归一化正浮点数{% raw %}$$UFL=2^l=2^{-1022}\approx2.2\times10^{-308}$${% endraw %}
* 最大归一化正浮点数{% raw %}$$OFL=(1-2^{-p})2^{u+1}=(1-2^{-53})2^{1024}\approx1.8\times10^{308}$${% endraw %}

同理，为了避免存储符号位，exponent实际存储为exponent+1023

#### __3.2.3__ 特殊值(corner cases)
本节主要讨论对于上文中提到的保留幂指数的使用，指如下几种用途：
* Singed zero：0
* Subnormal numbers：欠归一化浮点数(第一位有效数字为0)
* Infinities：无穷
* NaN：非数字(Not a Number)

以单精度数为例，用下图所示规则表示：
![single fp present](/assets/single-fp-present.png)

所以对于单精度数而言，能表示的最小正浮点数(欠归一化浮点数)为：

{% raw %}$$2^{-23}\times2^{-126}=2^{-149}\approx1.4\times10^{-45}$${% endraw %}

#### __3.2.3.1__ Signed zero
在协议中，0是有符号的，存在+0和-0，且+0=-0，但是对于某些操作运算(0点附近不连续的运算)，这两者的返回结果却不相等，比如1/(+0)=+infinity，1/(-0)=-infinity，类似的还有log、sign等运算

#### __3.2.3.2__ Subnormal numbers
填补了underflow的gap

#### __3.2.3.3__ Infinities
IEEE 754要求无穷数以合理的方式被处理，比如：
* {% raw %}$$(+\infty)+(+7)=(+\infty)$${% endraw %}.
* {% raw %}$$(+\infty)\times(-7)=(-\infty)$${% endraw %}.
* {% raw %}$$(+\infty)\times0=NaN$${% endraw %}.

#### __3.2.3.4__ NaNs
IEEE 754指定特殊值表示“非数字”，一般用于不合法运算，如0/0，sqrt(-1)等。NaNs分为两种：
* quiet NaNs(默认不做处理)
* signaling NaNs(抛出异常信息)

从NaNs的表示中可以看到未被使用的bit，这些一般作如下用途：
* 表示错误的信息和来源，但是如何表示并没有统一的标准。
* 作为浮点数系统的扩展，比如表示未被初始化的变量、用作其他特殊值。

### __3.3__ 基本运算
#### __3.3.1__ 舍入模式(Rounding modes)
因为浮点数有限的精度限制，导致绝大部分数不能准确用浮点数表示，只能用附近相邻的数近似表示，比如对于精度为8的浮点数，123456789只能被表示成{% raw %}$$1.2345678\times10^8$${% endraw %}或者{% raw %}$$1.2345679\times10^8$${% endraw %}，这类情况也适用于该浮点数系统中的循环小数（取决于该浮点数系统的base）。

对于任意一个数：
* 如果能用该浮点数系统准确表示，则误差{% raw %}$$\epsilon=0$${% endraw %}
* 否则，误差{% raw %}$$\epsilon\le\frac{ULP}{2}$${% endraw %}，其中ULP(unit in last place)表示该exponent下两个相邻浮点数的间隔大小。

所以对于近似表示的数，误差也会随着exponent的增大呈指数级增加。

> Q：既然对于绝大部分数，浮点数的表示都会存在误差，那如何保证其数学运算的正确性呢 ？
>
> A：IEEE 754协议要求正确的舍入(correct rounding)：先假设在无限精度的情况下的情况下进行运算，然后将结果进行相应精度的舍入。

协议中定义了如下几种舍入模式：
* roundTiesToEven：找最近的浮点数表示，若存在两个最近的，取最后一位有效数字为偶数(0)的浮点数。
* roundTiesToAway：找最近的浮点数表示，若存在两个最近的，取有较大绝对值的浮点数。
* roundTowardPositive：取更靠近正无穷大的的浮点数。
* roundTowardNegative：取更靠近负无穷大的浮点数。
* roundTowardZero：取更靠近0的浮点数

默认以及目前最常用的舍入模式是roundTiesToEven。

#### __3.3.2__ 加法&减法
小数加减法一般是对齐小数点，整数和小数部分分别相加减，该进位的进位，该借位的就借位。

浮点数加减法的处理一般分如下2步：
1. exponent对齐(实质是浮点数到定点数的转换)
2. 有效数字相加减
3. 对结果根据精度做舍入处理

下面以b=10，p=7的加法为例：
```
  e=5; s=1.234567 (123456.7)
+ e=2; s=1.017654 (101.7654)
```
```
  e=5; s=1.234567
+ e=5; s=0.001017654
------------------------
  e=5; s=1.235584654 (true sum)
  e=5; s=1.235585 (after rounding)
```
在存入结果时，需要对结果进行舍入操作处理，因此上面的结果有3位精度的误差。

#### __3.3.3__ 乘法&除法
乘除法的处理主要分如下3步：
1. 有效数字相乘除，指数部分相加减
2. 对结果根据精度做舍入处理
3. 结果归一化

仍以b=10，p=7举例如下：
```
  e=3; s=4.734612
x e=5; s=5.417242
--------------------------
  e=8; s=25.648538980104 (true product)
  e=8; s=25.64854 (after rounding)
  e=9; s=2.564854 (after normalization)
```

#### __3.3.4__ 误差
通常来讲，浮点数的相关的运算都有可能存在误差，而且可能性还比较大。根本原因是浮点数的离散分布的特点，且随着exponent增加，间隔指数级增加。一般误差的来源有两部分:
1. 实数用浮点数表示的时候可能引入误差
2. 运算操作可能引入误差，比如加减乘除

***
# __4__ 相关话题
1. 二进制和十进制浮点数的互转算法:[double conversion](https://github.com/google/double-conversion)
2. 如何计算任意精度的无理数，比如{% raw %}$$\sqrt2$${% endraw %}?
3. 数值分析中的误差分析

***
# __5__ 引用参考
* [IEEE Standard for Floating-Point Arithmetic 2008](/assets/IEEE_2008.pdf)
* [Floating-point arithmetic wikipedia page](https://en.wikipedia.org/wiki/Floating-point_arithmetic)
* [Single-precesion floating-point format wikipedia page](https://en.wikipedia.org/wiki/Single-precision_floating-point_format)
* [Floating Point Representation](https://courses.grainger.illinois.edu/cs357/fa2020/notes/ref-4-fp.html)
* [Floating Point Arthmetic: Issues and Limitations for Python](https://docs.python.org/3/tutorial/floatingpoint.html)
* [List of LaTeX symbols](https://latex.wikia.org/wiki/List_of_LaTeX_symbols)
* [Markdown syntax](https://daringfireball.net/projects/markdown/)
