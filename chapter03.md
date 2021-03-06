
# 第三讲：线性回归的概率解释、局部加权回归、逻辑回归

我们先来回顾一下上一讲中的内容。

* 我们用$\left(x^{(i)},\ y^{(i)}\right)$表示训练集中的第$i$个样本；
* 用向量$\vec\theta=(\theta_0,\ \cdots,\ \theta_n)$表示特征值的系数向量；
* 接着用$h_\theta\left(x^{(i)}\right)$表示用于拟合训练集中样本的假设函数（假设函数基于参数$\vec\theta$），并定义$h_\theta\left(x^{(i)}\right)=\displaystyle\sum_{j=0}^n\theta_jx_j^{(i)}=\theta^Tx$，$j$表示训练集中各样本的特征值$(x_1,\ \cdots,\ x_n)$的序号，共有$n$个特征值，而$x_0=1$，此为常数项$\theta_0$（即截距项）；
* 最后定义了成本函数$J(\theta)=\frac{1}{2}\left(\displaystyle\sum_{i=1}^mh_\theta\left(x^{(i)}\right)-y^{(i)}\right)^2$，其中$m$表示训练集中共有$m$个训练样本，其形式化描述为$\theta=\left(X^TX\right)^{-1}X^Ty$。

下面我们将接着上一讲的内容，从概率解释的角度介绍最小二乘的合理性。

## 3. 线性模型的概率解释

当我们遇到一个回归问题时，可能会有疑问，为什么使用线性回归？更具体的可能会问，为什么使用最小二乘成本函数$J$？这样合理吗？有那么多别的指标，比如预测值与实际值之差的绝对值、四次方等，为什么偏偏选择差的平方作为优化指标？在这一节我们将从一系列基于概率的假设中推出最小二乘回归的合理性。

仍旧使用“公寓租金”例子，我们假设价格是一些特征的线性函数加上$\epsilon^{(i)}$，于是有下面的式子将目标值与输入向量联系起来：

$$y^{(i)}=\theta^Tx^{(i)}+\epsilon^{(i)}$$

式中的$\epsilon^{(i)}$是误差项，它捕获了可能的未建模影响因素（比如我们的回归算法并没有纳入某个影响公寓租金的关键因素），也可能包含了存在于样本中的随机噪音。我们做出进一步假设，$\epsilon^{(i)}$是独立同分布（IDD: independently and identically distributed）的，服从期望为$0$方差为$\sigma^2$的高斯分布（或自然分布，Gaussian distribution/Normal distribution），即$\epsilon^{(i)}\sim N\left(0,\sigma^2\right)$，即$\epsilon^{(i)}$的概率密度函数为：

$$p\left(\epsilon^{(i)}\right)=\frac{1}{\sqrt{2\pi}\sigma}\exp\left(-\frac{\left(\epsilon^{(i)}\right)^2}{2\sigma^2}\right)$$

则可以得到：

$$p\left(y^{(i)}\mid x^{(i)};\theta\right)=\frac{1}{\sqrt{2\pi}\sigma}\exp\left(-\frac{\left(y^{(i)}-\theta^Tx^{(i)}\right)^2}{2\sigma^2}\right)$$

（这里教授简要说明了为什么使用高斯分布，一个原因是因为高斯分布便于计算；另一个原因是在处理实际问题的过程中，如果我们使用线性回归让后尝试测量误差的分布，大多情况下误差都服从高斯分布。高斯模型对于这类对于这类回归问题中的误差来说是一个很好的假设。另外，许多随机变量之和趋于服从高斯分布，如果误差是有许多效应共同导致的，例如卖家的情绪、买家的情绪、房子是否有花园、房间的壁炉数量等等，所有这些效应都是独立的，那么根据中心极限定理，这些效应的总和会近似的服从高斯分布。）

$p\left(y^{(i)}\mid x^{(i)};\theta\right)$表示以$\theta$为参数的对于给定$x^{(i)}$下$Y$的条件概率。我们不能将$\theta$也纳入条件（即不能记为$p\left(y^{(i)}\mid x^{(i)},\theta\right)$，这表示$y$是以$\theta,\ X$的为条件的概率，属于贝叶斯学派的观点），因为$\theta$并不是一个随机变量（频率学派的观点，我们不把$\theta$当做随机变量，我们认为$\theta$确有其值，但是我们现在不知道是多少）。我们也可以将$y^{(i)}$的分别表示为$y^{(i)}\mid x^{(i)};\theta\sim N\left(\theta^Tx^{(i)},\sigma^2\right)$。（即在给定$x^{(i)}$下假设函数的期望是$\theta^Tx^{(i)}$。）

现在，已知$X$（包含所有训练样本$x^{(i)}$的设计矩阵）和$\theta$，求$y^{(i)}$的分布。概率由$p\left(\vec y\mid X;\theta\right)$给出，这个量通常被视为一个关于$\vec y$（也可能关于$X$）的函数，有着固定参数$\theta$。现在我们明确的将这个量视为一个关于$\theta$的函数，我们仍叫它**似然函数（likelihood function）**（$\theta$的似然性是关于给定$X$下数据$Y$以$\theta$作为参数的概率，虽然$\theta$的似然性与数据$Y$的概率是一样的，而且尽管似然性和概率几乎是一样的意义，然而我们在这里使用似然性这个词是想强调，我们想要把$p\left(\vec y\mid X;\theta\right)$看做是在$\vec y,\ X$确定时关于$\theta$的函数，以后我们也会继续使用“参数的似然性”和“数据的概率”这种称谓，而不是“数据的似然性”和“参数的概率”这种说法。）：

$$L(\theta)=L\left(\theta;X,\vec y\right)=p\left(\vec y\mid X;\theta\right)$$

利用$\epsilon^{(i)}$独立的假设（也因为$y^{(i)}$是对于给定$x^{(i)}$的），上式可以写为：

$$\begin{align}L(\theta)&=\prod_{i=1}^mp\left(y^{(i)}\mid x^{(i)};\theta\right)\\&=\prod_{i=1}^m\frac{1}{\sqrt{2\pi}\sigma}\exp\left(-\frac{\left(y^{(i)}-\theta^Tx^{(i)}\right)^2}{2\sigma^2}\right)\end{align}$$

我们现在推出了将$x^{(i)}$与$y^{(i)}$联系在一起的概率模型，接下来需要考虑如何合理的对参数$\theta$做出猜测？最大似然估计告诉我们，应该选择使数据概率尽可能大的$\theta$，于是，我们应该选择能够使$L(\theta)$取最大值的$\theta$。

比起最大化$L(\theta)$，我们也可以选择最大化任意关于$L(\theta)$的严格增函数。因此，为了便于求导，我们可以最大化**对数似然估计（log likelihood）**$\mathscr{l}(\theta)$：

$$\begin{align}\mathscr{l}(\theta)&=\log L(\theta)\\&=\log \prod_{i=1}^m\frac{1}{\sqrt{2\pi}\sigma}\exp\left(-\frac{\left(y^{(i)}-\theta^Tx^{(i)}\right)^2}{2\sigma^2}\right)\\&=\sum_{i=1}^m\frac{1}{\sqrt{2\pi}\sigma}\exp\left(-\frac{\left(y^{(i)}-\theta^Tx^{(i)}\right)^2}{2\sigma^2}\right)\\&=m\log \frac{1}{\sqrt{2\pi}\sigma}-\frac{1}{\sigma^2}\cdot\frac{1}{2}\sum_{i=1}^m\left(y^{(i)}-\theta^Tx^{(i)}\right)^2\end{align}$$

于是，我们发现，最大化$\mathscr{l}(\theta)$的过程变成了最小化$\frac{1}{2}\sum_{i=1}^m\left(y^{(i)}-\theta^Tx^{(i)}\right)^2$，这个式子正是成本函数$J(\theta)$的定义式。

综上，可以看出，在关于训练数据的概率假设中，最小二乘回归与$\theta$的最大似然估计一致。也正是因为最小二乘回归其实是再做最大似然估计，所以我们才会强调最小二乘是一种“很自然”的方法。（不过，概率假设并不是证明“最小二乘是一种非常易用的、合理的求解过程”这一结论的*必要条件*，这只是众多假设中的一种，最小二乘在这种假设下合理，除此之外还有其他假设也可以证明这一结论。）

还需要注意的是，我们对$\theta$的求解并不依赖与$\sigma^2$，即使不知道$\sigma$我们依然完成了运算。我们将在后面关于指数族和一般线性模型的课程中使用这一结论。

## 欠拟合和过拟合

继续使用上一讲的例子，当我们考虑使用特征变量$x\in\mathbb{R}$来预测$y$，下图最左为使用直线$y=\theta_0+\theta_1x$拟合训练集，可以发现得到的直线并不能较为准确的描述训练数据的形态，我们说这不是一个良好的拟合。

<img src="./resource/chapter03_image01.png" width="800" alt="" align=center />

如果我们再加入一个特征值$x^2$，然后对$y=\theta_0+\theta_1x+\theta_2x^2$做拟合，于是我们得到一个稍好的拟合，如中图所示。看上去似乎是特征值取的越多越好，其实不然，取的过多也会有问题。如图中最右所示，这是用五阶多项式$y=\displaystyle\sum_{j=0}^5\theta_jx_j$得到的结果，可以看出曲线恰好能够经过每一个训练样本，但这并不是我们期望的良好的拟合，比如在“公寓租金”的例子中，这个曲线并不能良好的反映面积$x$对租金$y$的影响。我们将在后面的课程中形式化定义这个现象，并告诉大家如何鉴定什么才是一个“良好的”假设，现在大家只需要了解，左图叫做**欠拟合（underfitting）**，表示数据的特征并没有被拟合曲线良好的表达出来；而右图叫做**过拟合（overfitting）**。

从上面的例子，我们可以清楚的认识到，对特征的选择会在很大程度上影响学习算法的表现。（当学到模型选择时，我们也会见到自动选择“好”特征值的算法。）

## 4. 局部加权回归（LWR: locally weighted linear regression）

如果我们有足够的训练数据，特征值的选择就没那么重要了。在前面学到的线性回归中，为了查询$x$（即得到$h(x)$），我们将执行下面的步骤：

1. 寻找使$\sum_i\left(y^{(i)}-\theta^Tx^{(i)}\right)^2$取最小的$\theta$；
2. 给出预测结果$\theta^Tx$。

相比上面的过程，局部加权回归执行以下步骤：

1. 针对给定的查询点$x$，寻找使$\sum_i\omega^{(i)}\left(y^{(i)}-\theta^Tx^{(i)}\right)^2$取最小的$\theta$；
2. 给出预测结果$\theta^Tx$。

这里的$\omega$称为权值，是一个非负数。直观上看，如果对于某个$i$，$\omega^{(i)}$取值较大，则在计算$\theta$取值时，我们将尽可能的减小$\left(y^{(i)}-\theta^Tx^{(i)}\right)^2$项的取值（精确拟合）；反之，如果$\omega^{(i)}$取值较小，则$\left(y^{(i)}-\theta^Tx^{(i)}\right)^2$所得到的的误差项将足够小而忽略不计。

一种较为合理的权值设置（这里我们使用指数衰减函数，exponential decay function，权值的设置还有很多其他选择）：

$$\omega^{(i)}=exp\left(-\frac{\left(x^{(i)}-x\right)^2}{2\tau^2}\right)$$

易看出，对于给定的查询点$x$，权值的大小与训练样本$x^{(i)}$相对于$x$的位置密切相关：

* 如果$x^{(i)}$距离$x$很近，则$\omega^{(i)}$将取靠近$1$的值；
* 如果$x^{(i)}$距离$x$很远，则$\omega^{(i)}$将取到$0$附近。

因此，通过这种方法得到的$\theta$将更注重查询点$x$附近的训练样本（权重较高）的精确拟合，而忽略那些距离较远的训练样本（权重较低）。（尽管权值函数的样子看起来很像高斯分布的概率密度函数，但是这个函数跟高斯分布并没有直接联系，另外$\omega^{(i)}$并不是随机变量，也并不服从高斯分布，它只是一个样子恰好类似钟形的曲线。）

函数中的$\tau$称作**带宽（bandwidth）**（或波长）参数，它控制了权值随距离下降的速率。如果$\tau$取值较小，则会得到一个较窄的钟形曲线，这意味着离给定查询点$x$较远的训练样本$x^{(i)}$的权值（对查询点$x$附近训练样本的拟合的影响）将下降的非常快；而$\tau$较大时，则会得到一个较为平缓的曲线，于是查询点附近的训练样本的权重随距离而下降的速度就会相对比较慢。

值得注意的是，这是一个非参数算法，我们在使用这个方法时，是针对给定的查询点$x$进行计算，也就是每当我们对于一个给定的$x$做出预测时，都需要根据整个训练集重新进行拟合运算，如果训练集很大而查询很频繁的话，这个算法的代价将非常高。关于提高这个算法的效率，可以参考Andrew Moore关于KD-Tree的工作。

# 第二部分：分类问题（Classification）和逻辑回归（Logistic regression）

在前面的回归问题中，我们尝试预测的变量$y$是连续变量，现在我们来讨论分类问题。分类问题与回归问题不同之处在于，$y$的取值是少量的离散值。现在，我们先介绍**二元分类（binary classification）**，也就是$y$只能取$0$或$1$。（在二元分类中介绍的很多方法也可以推广至多元情况。）比如，我们尝试实现一个垃圾邮件分类器，用$x^{(i)}$表示邮件的某些特征，通过分类算法预测得到：如果该邮件是垃圾邮件时$y=1$，反之$y=0$。我们也把$0$称为**negative class**，把$1$称为**positive class**，我们有时也使用$-,\ +$作为标记。对于给定的$x^{(i)}$，$y^{(i)}$也称作训练样本的**label**。

## 5. 逻辑回归

对于分类问题，我们也可以无视$y$是取离散值的特征而继续使用线性回归，使用老办法通过$x$预测$y$的取值。然而，强行使用线性回归的方法处理分类问题通常会得到很糟糕的预测结果。况且，在我们已经明确知道$y\in\{0,\ 1\}$的情况下仍旧使用线性回归的$h_\theta(x)\in\mathbb{R}$是不合逻辑的。

所以，我们应该对原来的假设函数做出修改：

$$h_\theta(x)=g\left(\theta^Tx\right)=\frac{1}{1+e^{-\theta^Tx}}$$

这里的$g(z)=\frac{1}{1+e^{-z}}$，也叫作**逻辑函数（logistic function）**或**S型函数（sigmoid function）**，图像如下图所示：

<img src="./resource/chapter03_image02.png" width="400" alt="" align=center />

在$z\to\infty$时函数$g(z)$趋近于$1$，而$z\to-\infty$是函数$g(z)$趋近于$0$。而且不光是$g(z)$，包括上面的$h(x)$都在$(0, 1)$之间取值。有$\theta^Tx=\theta_0+\displaystyle\sum_{j=0}^n\theta_jx_j$，这里我们依然令$x_0=1$。

其它的在$(-\infty,\ \infty)$上能够从$0$取到$1$的可导的连续函数也可以使用，但是因为一些原因（我们将在后面的关于GLM、生成学习法的课程中了解），选择上面的逻辑函数是一种“自然”的结果。再继续前，我们先来看一下逻辑函数求导的优良性质：

$$\begin{align}g'(z)&=\frac{\mathrm d}{\mathrm dz}\frac{1}{1+e^{-z}}\\&=\frac{1}{\left(1+e^{-z}\right)^2}\left(e^{-z}\right)\\&=\frac{1}{1+e^{-z}}\cdot\left(1-\frac{1}{1+e^{-z}}\right)\\&=g(z)(1-g(z))\end{align}$$

接下来，我们应该如何用$\theta$拟合逻辑回归模型呢？在线性回归中我们知道了，从一些列假设下的最大似然估计可以推导出最小二乘回归的合理性，所以，我们也可以按照这个思路，给我们的分类模型赋予一些列概率假设，然后通过最大似然估计拟合参数。假设：

$$\begin{align}P(y=1\mid x;\theta)&=h_\theta(x)\\P(y=0\mid x;\theta)&=1-h_\theta(x)\end{align}$$

当然，我们也可以将这两个假设合入一个简洁的式子：

$$p(y\mid x;\theta)=(h_\theta(x))^y(1-h_\theta(x))^{(1-y)}$$

假设训练集中的$m$个训练样本是相互独立的，我们就可以这样写出参数的似然估计：

$$\begin{align}L(\theta)&=p\left(\vec y\mid X;\theta\right)\\&=\prod_{i=1}^m p\left(y^{(i)}\mid x^{(i)};\theta\right)\\&=\prod_{i=1}^m\left(h_\theta\left(x^{(i)}\right)\right)^{y^{(i)}}\left(1-h_\theta\left(x^{(i)}\right)\right)^{1-y^{(i)}}\end{align}$$

跟线性回归中的运算一样，我们取对数便于求导：

$$\begin{align}\mathscr{l}(\theta)&=\log L(\theta)\\&=\sum_{i=1}^my^{(i)}\log h_\theta\left(x^{(i)}\right)+\left(1-y^{(i)}\right)\log\left(1-h_\theta\left(x^{(i)}\right)\right)\end{align}$$

我们依然沿用线性回归中的思路，通过求导发现最大化似然函数的极值点，这次我们使用梯度上升法。我们使用矢量记法，则有更新规则为$\theta:=\theta+\alpha\nabla_\theta\mathscr{l}(\theta)$。（因为想要求函数的最大值，所以我们在更新规则里使用了加号。）我们现在假设训练集中只有一个训练样本$(x,\ y)$，对其求导，希望能导出适用于随机梯度上升更新规则：

$$\begin{align}\frac{\partial}{\partial\theta_j}\mathscr{l}(\theta)&=\left(y\frac{1}{g\left(\theta^Tx\right)}-(1-y)\frac{1}{1-g\left(\theta^Tx\right)}\right)\frac{\partial}{\partial\theta_j}g\left(\theta^Tx\right)\\&=\left(y\frac{1}{g\left(\theta^Tx\right)}-(1-y)\frac{1}{1-g\left(\theta^Tx\right)}\right)g\left(\theta^Tx\right)\left(1-g\left(\theta^Tx\right)\right)\frac{\partial}{\partial\theta_j}\theta^Tx\\&=\left(y\left(1-g\left(\theta^Tx\right)\right)-(1-y)g\left(\theta^Tx\right)\right)x_j\\&=(y-h_\theta(x))x_j\end{align}$$

在上面的求导中，从第一步到第二步我们使用了逻辑函数求导性质$g'(z)=g(z)(1-g(z))$。这样，我们也就得到了适用于随机梯度上升的更新规则：

$$\theta_j:=\theta_j+\alpha\left(y^{(i)}-h_\theta\left(x^{(i)}\right)\right)x_j$$

如果使用批量梯度上升则是：

$$\theta_j:=\theta_j+\alpha\sum_{i=1}^m\left(y^{(i)}-h_\theta\left(x^{(i)}\right)\right)x_j$$

回顾上一讲的最小均方法法，我们会发现上面这条更新规则和以前的更新规则一模一样，我们在这里必须说明，$h_\theta(x)$已经变为关于$\theta^Tx^{(i)}$的非线性函数，所以这并不是同一个算法。尽管如此，对于不同的算法得到同样形式的更新规则，我们还是感到很惊讶。对于这个现象，我们会在后面关于GLM模型的课程中给出解释。

## 6. 感知算法（perceptron learning algorithm）

最后，我们来简要的介绍一下感知算法，我们以后在介绍学习理论是还会继续讨论这个算法。对于上面的逻辑回归，我们如何“强迫”算法只输出$0$或$1$？我们会自然的想到改变$g$的定义，使其变为一个阈函数：

$$g(z)=\begin{cases}1\quad z\geq 0\\0\quad z\lt 0\end{cases}$$

如果我们依然令$h_\theta(x)=g\left(\theta^Tx\right)$，并将$g$修改为上面的阈函数，然后使用$\theta_j:=\theta_j+\alpha\displaystyle\sum_{i=1}^m\left(y^{(i)}-h_\theta\left(x^{(i)}\right)\right)x_j$或$\theta_j:=\theta_j+\alpha\left(y^{(i)}-h_\theta\left(x^{(i)}\right)\right)x_j$作为学习规则，那么我们就实现了一个**感知算法（perceptron learning algorithm）**。

在六十年代，这个感知算法被认为是一种粗略的描述脑中独立神经元工作方式的模型。该模型很简单，所以我们也将它作为日后讨论学习理论的起点。需要注意的是，尽管这个算法看起来并不特殊，但实际上，与其前面的逻辑回归、最小二乘线性回归比起来，这是一个非常不同的算法：比如，我们很难赋予它一个在概率上有意义的解释，也很难从最大似然估计推导出感知算法。
