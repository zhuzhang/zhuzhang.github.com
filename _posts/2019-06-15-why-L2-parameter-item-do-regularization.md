---
layout: post
title: Deep Learning中的L2范数是如何做正则化的？
---

训练模型，通常会遇到三种情况：
1. 欠拟合：没有获得数据的真实模型
2. 拟合：刚好获得数据的真实模型
3. 过拟合：不只获得数据的真实模型，还包含了其他的数据模型（训练集的噪声）

正则化的目的就是将上述情况3转化成2。
那正则项 {% raw %} $$\Omega(\boldsymbol{\theta})=\frac{1}{2}\|\boldsymbol{\omega}\|^{2}_{2}$$ {% endraw %} 是如何达到这个目的的呢？ 

令需要优化的目标函数为：
{% raw %}
$$\widetilde{J}(\boldsymbol{\omega};\textbf{X},\textbf{y})=\frac{\alpha}{2}\boldsymbol{\omega}^{T}\boldsymbol{\omega}+J(\boldsymbol{\omega};\textbf{X},\textbf{y})$$
{% endraw %}
则其关于{% raw %}$$\boldsymbol{\omega}$${% endraw %}的梯度为：
{% raw %}
$$\nabla_{\boldsymbol{\omega}}\widetilde{J}=\alpha\boldsymbol{\omega}+\nabla_{\boldsymbol{\omega}}J$$
{% endraw %}
令学习率为{% raw %}$$\epsilon$${% endraw %}，则参数更新式为：
{% raw %}
$$\boldsymbol{\omega}\gets\boldsymbol{\omega}-\epsilon(\alpha\boldsymbol{\omega}+\nabla_{\boldsymbol{\omega}}J)$$
{% endraw %}
即为：
{% raw %}
$$\boldsymbol{\omega}\gets(1-\epsilon\alpha)\boldsymbol{\omega}-\epsilon\nabla_{\boldsymbol{\omega}}J$$
{% endraw %}
由此可以看出，参数单步更新在之前的基础上有个衰减因子{% raw %}$$\epsilon\alpha$${% endraw %}的影响(weight decay)。

那在整个训练过程中，这个正则项是如何影响训练结果的呢？下面我们可以尝试定性分析下：

利用目标函数{% raw %}$$J$${% endraw %}在{% raw %}$$\boldsymbol{\omega}^*=argmin_{\boldsymbol{\omega}}J(\boldsymbol{\omega})$${% endraw %}处的二阶泰勒展开式逼近目标函数：
{% raw %}
$$\widehat{J}(\boldsymbol{\omega})=J(\boldsymbol{\omega}^*)+(\boldsymbol{\omega}-\boldsymbol{\omega}^*)^T\boldsymbol{G}+\frac{1}{2}(\boldsymbol{\omega}-\boldsymbol{\omega}^*)^T\boldsymbol{H}(\boldsymbol{\omega}-\boldsymbol{\omega}^*)$$
{% endraw %}
其中{% raw %}$$\boldsymbol{G}$${% endraw %}为{% raw %}$$J(\boldsymbol{\omega})$${% endraw %}在{% raw %}$$\boldsymbol{\omega}^*$${% endraw %}处的梯度向量，{% raw %}$$\boldsymbol{H}$${% endraw %}为{% raw %}$$J(\boldsymbol{\omega})$${% endraw %}在{% raw %}$$\boldsymbol{\omega}^*$${% endraw %}处的Hessian矩阵

由于{% raw %}$$J(\boldsymbol{\omega})$${% endraw %}在{% raw %}$$\boldsymbol{\omega}^*$${% endraw %}处取得最小值，故{% raw %}$$\boldsymbol{G}=0$${% endraw %}，且{% raw %}$$\boldsymbol{H}$${% endraw %}为半正定矩阵

所以要使{% raw %}$$\widehat{J}(\boldsymbol{\omega})$${% endraw %}最小，则有：
{% raw %}
$$\nabla_{\boldsymbol{\omega}}\widehat{J}(\boldsymbol{\omega})=\boldsymbol{H}(\boldsymbol{\omega}-\boldsymbol{\omega}^*)=0$$
{% endraw %}

现在考虑L2正则项，这时近似的目标函数{% raw %}$$\widehat{J}(\boldsymbol{\omega})$${% endraw %}的梯度变为：
{% raw %}
$$\nabla_{\boldsymbol{\omega}}\widehat{J}(\boldsymbol{\omega})=\alpha\boldsymbol{\omega}+\boldsymbol{H}(\boldsymbol{\omega}-\boldsymbol{\omega}^*)=0$$
{% endraw %}
其解为：
{% raw %}
$$\widehat{\boldsymbol{\omega}}=(\alpha\boldsymbol{I}+\boldsymbol{H})^{-1}\boldsymbol{H\omega^*}$$
{% endraw %}
如果{% raw %}$$\alpha=0$${% endraw %}，则{% raw %}$$\widehat{\boldsymbol{\omega}}=\boldsymbol{\omega}^*$${% endraw %}。

如果{% raw %}$$\alpha\gt0$${% endraw %}，令{% raw %}$$\boldsymbol{H}=\boldsymbol{Q}\boldsymbol{\Lambda}\boldsymbol{Q}^T$${% endraw %}，则有：
{% raw %}
$$\widehat{\boldsymbol{\omega}}=\boldsymbol{Q}(\boldsymbol{\Lambda}+\alpha\boldsymbol{I})^{-1}\boldsymbol{\Lambda}\boldsymbol{Q}^T\boldsymbol{\omega}^*$$
{% endraw %}
由此可以看出，L2正则项的效果是让参数{% raw %}$$\boldsymbol{\omega}$${% endraw %}沿着{% raw %}$$\boldsymbol{H}$${% endraw %}的特征向量方向进行比例放缩，其放缩比例为{% raw %}$$\frac{\lambda_i}{\lambda_i+\alpha}$${% endraw %}（{% raw %}$$\lambda_i$${% endraw %}为{% raw %}$$\boldsymbol{H}$${% endraw %}的第i个特征值）。

当{% raw %}$$\lambda_i\gg\alpha$${% endraw %}时，{% raw %}$$\frac{\lambda_i}{\lambda_i+\alpha}\rightarrow1$${% endraw %}。

当{% raw %}$$\lambda_i\ll\alpha$${% endraw %}时，{% raw %}$$\frac{\lambda_i}{\lambda_i+\alpha}\rightarrow0$${% endraw %}。

所以对于目标函数优化影响越小的参数，其衰减效果越明显。

这里面有个假设是:对目标函数优化影响不大的参数是由于训练集的噪声引起的。

所以去除噪声，增加模型的信噪比，正是正则化的目标。 

附效果示例图：
![示例图](/assets/l2-regularize.png)

# References

Bengio, Yoshua, Ian J. Goodfellow, and Aaron Courville. "Deep learning."


