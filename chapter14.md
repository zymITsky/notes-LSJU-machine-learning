
# 第十四讲：因子分析的EM算法、主成分分析

先来回顾一下上一讲推导出的结论，对于随机变量$x=\begin{bmatrix}x_1\\x_2\end{bmatrix},\ x\sim\mathcal N(\mu,\varSigma)$，其中$\mu=\begin{bmatrix}\mu_1\\\mu_2\end{bmatrix},\ \varSigma=\begin{bmatrix}\varSigma_{11}&\varSigma_{12}\\\varSigma_{21}&\varSigma_{22}\end{bmatrix}$，则有：

* $x_1$的边缘分布$p(x_1)$为：

$$
x\sim\mathcal N\left(\mu_1,\varSigma_{11}\right)
$$

* 在$x_2$条件下$x_1$的概率$p(x_1\mid x_2)$为：

$$
\begin{align}
x_1\mid x_2&\sim\mathcal N\left(\mu_{1\mid2},\varSigma_{1\mid2}\right)\\
\mu_{1\mid2}&=\mu_1+\varSigma_{12}\varSigma_{22}^{-1}(x_2-\mu_2)\tag{1}\\
\varSigma_{1\mid2}&=\varSigma_{11}-\varSigma_{12}\varSigma_{22}^{-1}\varSigma_{21}\tag{2}
\end{align}
$$

上一讲还得到了$z,x$的联合分布为：

$$
\begin{bmatrix}z\\x\end{bmatrix}\sim\mathcal N\left(\begin{bmatrix}\vec0\\\mu\end{bmatrix},\begin{bmatrix}I&\varLambda^T\\\varLambda&\varLambda\varLambda^T+\varPsi\end{bmatrix}\right)\tag{3}
$$

所以，随机变量$x$的边缘分布为$x\sim\mathcal N\left(\mu,\varLambda\varLambda^T+\varPsi\right)$，最终得到参数的对数似然函数：

$$
\mathscr l(\mu,\varLambda,\varPsi)=\log\prod_{i=1}^m\frac{1}{\sqrt{(2\pi)^n\left\lvert\varLambda\varLambda^T+\varPsi\right\rvert}}\exp\left(-\frac{1}{2}\left(x^{(i)}-\mu\right)^T\left(\varLambda\varLambda^T+\varPsi\right)^{-1}\left(x^{(i)}-\mu\right)\right)
$$

（这里之所以选择最大化关于$p\left(x^{(i)}\right)$的似然函数，而不是关于联合分布$p\left(x^{(i)},z^{(i)}\right)$的似然函数，是因为对于给定训练集$\left\{x^{(1)},\cdots,x^{(m)}\right\}$，我们只能观测到随机变量$x$，$z$只是潜在变量。我们需要计算的是$\displaystyle\operatorname*{max}_\theta\prod_{i=1}^mp\left(x^{(i)};\theta\right)=\operatorname*{max}_\theta\prod_{i=1}^m\int_{z^{(i)}}p\left(x^{(i)},z^{(i)};\theta\right)\mathrm dz^{(i)}$。）

我们无法直接求得上面这个式子的解析解，所以转而使用EM算法逼近其最优解。

## 4. 因子分析模型的EM算法

E步骤的推导比较简单，我们需要计算$Q_i\left(z^{(i)}\right)=p\left(z^{(i)}\mid x^{(i)};\mu,\varLambda,\varPsi\right)$，将$(1),(2)$式带入$(3)$式，计算条件概率$z^{(i)}\mid x^{(i)}$：

$$
\begin{align}
z^{(i)}\mid x^{(i)};\mu,\varLambda,\varPsi&\sim\mathcal N\left(\mu_{z^{(i)}\mid x^{(i)}},\varSigma_{z^{(i)}\mid x^{(i)}}\right)\\
\mu_{z^{(i)}\mid x^{(i)}}&=\varLambda^T\left(\varLambda\varLambda^T+\varPsi\right)^{-1}\left(x^{(i)}-\mu\right)\\
\varSigma_{z^{(i)}\mid x^{(i)}}&=I-\varLambda^T\left(\varLambda\varLambda^T+\varPsi\right)^{-1}\varLambda
\end{align}
$$

将$\mu_{z^{(i)}\mid x^{(i)}},\varSigma_{z^{(i)}\mid x^{(i)}}$带入高斯分布$Q_i\left(z^{(i)}\right)$得：

$$
Q_i\left(z^{(i)}\right)=\frac{1}{\sqrt{(2\pi)^k\left\lvert\varSigma_{z^{(i)}\mid x^{(i)}}\right\rvert}}\exp\left(-\frac{1}{2}\left(z^{(i)}-\mu_{z^{(i)}\mid x^{(i)}}\right)^T\varSigma_{z^{(i)}\mid x^{(i)}}^{-1}\left(z^{(i)}-\mu_{z^{(i)}\mid x^{(i)}}\right)\right)
$$

再来看M步骤，我们需要最大化下式关于参数$\mu,\varLambda,\varPsi$的函数：

$$
\sum_{i=1}^m\int_{z^{(i)}}Q_i\left(z^{(i)}\right)\log\frac{p\left(x^{(i)},z^{(i)};\mu,\varLambda,\varPsi\right)}{Q_i\left(z^{(i)}\right)}\mathrm dz^{(i)}\tag{4}
$$

（潜在随机变量$z$服从某个高斯分布，所以这里使用积分。）

我们这里只写出关于优化$\varLambda$的推导，其他参数的推导过程类似，$(4)$式可化简为：

$$
\begin{align}
\sum_{i=1}^m\int_{z^{(i)}}Q_i\left[\log p\left(x^{(i)}\mid z^{(i)};\mu,\varLambda,\varPsi\right)+\log p\left(z^{(i)}\right)-\log Q_i\left(z^{(i)}\right)\right]\mathrm dz^{(i)}\tag{5}\\
=\sum_{i=1}^m\operatorname*{E}_{z^{(i)}\sim Q_i}\left[\log p\left(x^{(i)}\mid z^{(i)};\mu,\varLambda,\varPsi\right)+\log p\left(z^{(i)}\right)-\log Q_i\left(z^{(i)}\right)\right]\tag{6}
\end{align}
$$

这一步化简是推导中的一个重要的技巧，将积分部分看做是随机变量$z^{(i)}$在一个高斯分布$Q_i$下关于某函数的期望（$\displaystyle\operatorname*{E}_{z^{(i)}\sim Q_i}$指的服从分布$Q_i$的随机变量$z^{(i)}$的期望，后面的推导中，我们会在表达不产生歧义时省略这个下标；另外，化简计算中用到了[联合、条件及边缘概率的关系](https://en.wikipedia.org/wiki/Joint_probability_distribution#Density_function_or_mass_function)$p(x\mid z)p(z)=p(x,z)$）。继续化简，删掉式中与$\varLambda$无关的项（$z$服从高斯分布$\mathcal N(0,I)$；而$Q_i\left(z^{(i)}\right)$虽然服从高斯分布$\mathcal N\left(\mu_{z^{(i)}\mid x^{(i)}},\varSigma_{z^{(i)}\mid x^{(i)}}\right)$，但在E步骤确定了$Q_i\left(z^{(i)}\right)$之后，这个分布就固定了，其中的参数不再起变量的作用，所以这两项均与$\mu,\varSigma,\varPsi$无关），则需要最大化的目标为：

$$
\begin{align}
\sum_{i=1}^m&\mathrm E\left[\log p\left(x^{(i)}\mid z^{(i)};\mu,\varLambda,\varPsi\right)\right]\\
=&\sum_{i=1}\mathrm E\left[\log\frac{1}{\sqrt{(2\pi)^n\lvert\varPsi\rvert}}\exp\left(-\frac{1}{2}\left(x^{(i)}-\mu-\varLambda z^{(i)}\right)^T\varPsi^{-1}\left(x^{(i)}-\mu-\varLambda z^{(i)}\right)\right)\right]\\
=&\sum_{i=1}^m\mathrm E\left[-\frac{1}{2}\log\lvert\varPsi\rvert-\frac{n}{2}\log(2\pi)-\frac{1}{2}\left(x^{(i)}-\mu-\varLambda z^{(i)}\right)^T\varPsi^{-1}\left(x^{(i)}-\mu-\varLambda z^{(i)}\right)\right]
\end{align}
$$

因为$x^{(i)}\mid z^{(i)}\sim\mathcal N\left(\mu+\varLambda z,\varPsi\right)$，最大化上面这个关于$\varLambda$的函数，只有最后一项与$\varLambda$有关，于是对$\varLambda$求偏导，并使用[第二讲](chapter02.ipynb)中提到的性质：$\forall a\in\mathbb R,\mathrm{tr}a=a$、$\mathrm{tr}AB=\mathrm{tr}BA$以及$\nabla_\varLambda\mathrm{tr}ABA^TC=CAB+C^TAB$这三条性质得到（$\varLambda$是对角矩阵）：

$$
\begin{align}
\nabla_\varLambda\sum_{i=1}^m-&\mathrm E\left[\frac{1}{2}\left(x^{(i)}-\mu-\varLambda z^{(i)}\right)^T\varPsi^{-1}\left(x^{(i)}-\mu-\varLambda z^{(i)}\right)\right]\\
=\sum_{i=1}^m&\nabla_\varLambda\mathrm E\left[-\mathrm{tr}\left(\frac{1}{2}\left(z^{(i)}\right)^T\varLambda^T\Psi^{-1}\varLambda z^{(i)}\right)+\mathrm{tr}\left(\left(z^{(i)}\right)^T\varLambda^T\Psi^{-1}\left(x^{(i)}-\mu\right)\right)\right]\\
=\sum_{i=1}^m&\nabla_\varLambda\mathrm E\left[-\mathrm{tr}\frac{1}{2}\varLambda^T\varPsi^{-1}\varLambda z^{(i)}\left(z^{(i)}\right)^T+\mathrm{tr}\varLambda^T\varPsi^{-1}\left(x^{(i)}-\mu\right)\left(z^{(i)}\right)^T\right]\\
=\sum_{i=1}^m&\mathrm E\left[-\varPsi^{-1}\varLambda z^{(i)}\left(z^{(i)}\right)^T+\varPsi^{-1}\left(x^{(i)}-\mu\right)\left(z^{(i)}\right)^T\right]
\end{align}
$$

将上面的结果置为零并化简得到：

$$
\sum_{i=1}^m\varLambda\operatorname*{E}_{z^{(i)}\sim Q_i}\left[z^{(i)}\left(z^{(i)}\right)^T\right]=\sum_{i=1}^m\left(x^{(i)}-\mu\right)\operatorname*{E}_{z^{(i)}\sim Q_i}\left[\left(z^{(i)}\right)^T\right]
$$

最后，解出$\varLambda$得到：

$$
\varLambda=\left(\sum_{i=1}^m\left(x^{(i)}-\mu\right)\operatorname*{E}_{z^{(i)}\sim Q_i}\left[\left(z^{(i)}\right)^T\right]\right)\left(\sum_{i=1}^m\operatorname*{E}_{z^{(i)}\sim Q_i}\left[z^{(i)}\left(z^{(i)}\right)^T\right]\right)^{-1}\tag{7}
$$

有趣的是，这个表达式和我们从最小二乘回归得出的正规方程组$\theta^T=\left(y^TX\right)\left(X^TX\right)^{-1}$有着相似之处，可以看出$x$是关于$z$（加上噪音项）的线性函数。在E步骤中猜测$z$之后，我们尝试估计$x$与$z$之间的线性关系$\varLambda$。这个相似点到没什么，但应该注意两个算法之间的重要不同——最小二乘法会对$z$进行“最佳猜测”，我们在后面会看到这个不同。

要完成M步骤的参数$\varLambda$更新，我们需要把$(7)$式计算出来。由$Q_i$的定义可知，它是一个以$\mu_{z^{(i)}\mid x^{(i)}}$为期望，$\varSigma_{z^{(i)}\mid x^{(i)}}$为方差的高斯分布，则有：

$$
\begin{align}
\operatorname*{E}_{z^{(i)}\sim Q_i}\left[\left(z^{(i)}\right)^T\right]&=\mu_{z^{(i)}\mid x^{(i)}}^T\\
\operatorname*{E}_{z^{(i)}\sim Q_i}\left[z^{(i)}\left(z^{(i)}\right)^T\right]&=\mu_{z^{(i)}\mid x^{(i)}}\mu_{z^{(i)}\mid x^{(i)}}^T+\varSigma_{z^{(i)}\mid x^{(i)}}
\end{align}
$$

上面的第二个式子可以从协方差的定义推出，对于随机变量$Y$，$\mathrm{Cov}(Y)=\mathrm E\left[YY^T\right]-\mathrm E[Y]\mathrm E[Y]^T$，$\mathrm E\left[YY^T\right]=\mathrm E[Y]\mathrm E[Y]^T+\mathrm{Cov}(Y)$，带回$(7)$式即可得到$\varLambda$的更新规则：

$$
\varLambda=\left(\sum_{i=1}^m\left(x^{(i)}-\mu\right)\mu_{z^{(i)}\mid x^{(i)}}^T\right)\left(\sum_{i=1}^m\mu_{z^{(i)}\mid x^{(i)}}\mu_{z^{(i)}\mid x^{(i)}}^T+\varSigma_{z^{(i)}\mid x^{(i)}}\right)^{-1}\tag{8}
$$

我们一定要知道等式右边$\varSigma_{z^{(i)}\mid x^{(i)}}$的意义，这是在$x^{(i)}$条件下$z^{(i)}$的后验分布$p\left(z^{(i)}\mid x^{(i)}\right)$的协方差，而M步骤必须将这个关于$z^{(i)}$的不确定项考虑在内。一个在EM推导中常见的错误就是认为在E步骤中只需要计算潜在随机变量的期望$\mathrm E[z]$，之后用它代替M步骤优化中出现的所有“$z$的期望”就可以完成了，比如上面第二项，将$\mathrm E\left[z^{(i)}\left(z^{(i)}\right)^T\right]$直接写成$\mu_{z^{(i)}\mid x^{(i)}}\mu_{z^{(i)}\mid x^{(i)}}^T$。虽然这样确实对简单的模型（比如混合高斯模型或混合贝叶斯模型）确实有效，但这是不正确的。在我们对因子分析模型的推导中，除了$\mathrm E[z]$之外还需要$\mathrm E\left[zz^T\right]$，而且我们在上面确实看到了$\mathrm E[z]$与$\mathrm E\left[zz^T\right]$之间相差了$\varSigma_{z\mid x}$项。因此，M步骤必须将$z$变量在后验分布$p\left(z^{(i)}\mid x^{(i)}\right)$上的协方差考虑在内。

最后，我们给出M步骤中其他参数$\mu,\varPsi$的更新规则，推导这些参数并不难，首先是参数$\mu$的更新规则：

$$
\mu=\frac{1}{m}\sum_{i=1}^mx^{(i)}
$$

参数$\mu$并不像其他变量那样在更新中变化（也就是说和参数$\varLambda$不同，式子右边并不依赖$Q_i\left(z^{(i)}\right)=p\left(z^{(i)}\mid x^{(i)};\mu,\varLambda,\varPsi\right)$，也就不依赖其他参数了），这个参数只需要计算一次，并且随着算法的执行，这个参数也不需要再做更新。类似的，对角矩阵$\varPsi$的更新规则为：

$$
\varPhi=\frac{1}{m}\sum_{i=1}^mx^{(i)}\left(x^{(i)}\right)^T-x^{(i)}\mu_{z^{(i)}\mid x^{(i)}}\varLambda^T-\varLambda\mu_{z^{(i)}\mid x^{(i)}}\left(x^{(i)}\right)^T+\varLambda\left(\mu_{z^{(i)}\mid x^{(i)}}\mu_{z^{(i)}\mid x^{(i)}}^T+\varSigma_{z^{(i)}\mid x^{(i)}}\right)\varLambda^T
$$

再令$\varPsi_{ii}=\varPhi_{ii}$即可得到对角矩阵$\varPsi$（也就是让$\varPsi$只取$\varPhi$的对角线元素）。

总结整个推导过程，其中有三个地方需要注意：
* 在E步骤中我们既要计算期望，又要计算协方差；
* 在M步骤中，将积分视为某个分布的期望可以极大的简化推导；
* 注意要将协方差考虑在内。

# 第十一部分：主成分分析（Principal Components Analysis）

在上一部分的讲解中，针对“$x\in\mathbb R^n$‘近似的’分布在某个$k$维子空间中，其中$k\ll n$”情形，我们使用了因子分析模型对其建模。在讲解中我们假设将某个高斯分布的潜在变量$z^{(i)}$映射在仿射空间$\left\{\varLambda z+\mu;z\in\mathbb R^k\right\}$，然后再加上噪音项$\varPsi$后，就能够得到训练集$\left\{x^{(1)},\cdots,x^{(m)}\right\}$。因子分析是一个基于概率模型的、使用EM算法做参数估计的模型。

接下来，我们将介绍主成分分析（PCA: Principal Components Analysis），该算法同样尝试确定训练样本所“近似”分布的子空间。不同的是，PCA更加直接，而且算法只需要计算特征向量（在MATLAB中使用`eig`命令即可得到），并不需要再应用EM算法估计参数。

假设有数据集$\left\{x^{(i)};i=i,\cdots,m\right\}$，数据集代表了$m$种不同类型的汽车的属性，比如它们的最大速度、转弯半径等等，并且有$x^{(i)}\in\mathbb R^n, n\ll m$。但是有两个属性我们不知道，$x_i,x_j$，它们分别是单位为英里/小时的汽车的最大速度，和单位为公里/小时的汽车最大速度。实际上这两个属性可以说是线性相关的，只不过可能在单位转换后近似取整时造成细微的差别（比如从kph转到mph）。因此，这个数据集可以近似的看做是在$n-1$维子空间中的。那么，我们如何使用算法自动的检测或者删除像上面这种冗余特征值呢？

上面的例子可能有一种“故意设计”的感觉，那么我们来看一个实际遇到的例子：假设数据集来自关于遥控直升机操纵者的问卷调查，其中$x_1^{(i)}$代表操纵者的飞行水平、$x_2^{(i)}$代表操纵者对飞行的热爱程度。由于遥控直升机很难操作，只有那些鉴定的、衷心热爱这项运动的人才能够成为好飞行员。因此，$x_1,x_2$是强相关的。实际上，我们可能会发现数据分布在某个坐标对角线附近（下图$u_1$方向），这反映出操纵者飞行水平与热爱程度的内在因果联系，数据集中可能会存在一个背离此对角线的较小的噪音（$u_2$方向）。我们怎样才能让算法自动计算这个$u_1$方向？

<img src="./resource/chapter14_image01.png" width="350" alt="" align=center />

接下来将介绍PCA算法，不过通常在PCA算法运行前，我们需要对数据进行预处理——对数据的期望及方差进行标准化：
1. 令$\displaystyle\mu=\frac{1}{m}\sum_{i=1}^mx^{(i)}$；
2. 将所有$x^{(i)}$替换为$x^{(i)}-\mu$；
3. 令$\displaystyle\sigma_j^2=\frac{1}{m}\sum_{i=1}^m\left(x_j^{(i)}\right)^2$；
4. 将所有$x_j^{(i)}$替换为$\displaystyle\frac{x_j^{(i)}}{\sigma_j}$。

第$(1-2)$步使数据期望归零，如果数据期望原本就是零（举个现实中的例子，比如演讲等声频信号关于时间的序列），则可以省略这两步。第$(3-4)$步会缩放样本的每个分量使其具有单位化的方差，这一步用来保证可以用相同的“尺度”对待不同的属性。举个例子，比如$x_1$是以mph表示的汽车最高时速（可能的取值为从几十到一百多），而$x_2$表示汽车座位数量（可能的取值为二到四），通过这个标准化步骤重新调整不同的属性，使它们具有可比性。如果我们具有的关于数据的先验知识表明不同的属性也在相同的尺度下，具有可比性，那么$(3-4)$步也可以省略。一个现实中的例子，比如每一个样本$x^{(i)}$表示一副灰阶图，其每个分量$x_j^{(i)}$表示第$i$幅图第$j$个像素的亮度，它们都会从$\{1,\cdots,255\}$取值，也就可以省略这个除以标准差的步骤了。

现在，标准化了数据集之后，如何计算“变量的主轴”$u$（即数据大致的分布方向）？其中一种办法是找到这样一个单位向量$u$，当我们把数据投影在$u$方向上时，投影数据的方差之和能够取最大值。直观的讲，就是数据一开始就包含某些方差/信息在内，而我们需要找到一个方向$u$，当把数据投影在由$u$指出的方向/子空间上时，能够尽可能的保留这些固有的方差/信息。

考虑下面的数据集，使用前面的标准化步骤：

<img src="./resource/chapter14_image02.png" width="350" alt="" align=center />

现在，假设我们选择的$u$给出的方向如下图所示，图中的圆点表示原始数在该直线上的投影。

<img src="./resource/chapter14_image03.png" width="350" alt="" align=center />

在上图中可以看到投影之后的数据依然具有很大的方差，而且远离零点。相反，如果我们按照下图选择投影直线的方向：

<img src="./resource/chapter14_image04.png" width="350" alt="" align=center />

此时，这些投影点的方差非常小，且距离原点更近。

所以，我们希望按照前一张图中的模式让算法自动选择$u$的方向。接下来我们就正式的表达这个计算过程，对于给定向量$u$和样本向量$x$，$x$向$u$做投影，投影的长度为$x^Tu$，也就是说如果$x^{(i)}$是数据集中的一个样本点（图中的×），那么它在$u$上的投影点（图中×对应的“黑点”）到原点的距离为$x^Tu$。因此，为了最大化投影长度的方差，我们需要选择一个单位向量$u$使得下面的式子取最大值：

$$
\begin{align}
\frac{1}{m}\sum_{i=1}^m\left(\left(x^{(i)}\right)^Tu\right)^2&=\frac{1}{m}\sum_{i=1}^mu^Tx^{(i)}\left(x^{(i)}\right)^Tu\\
&=u^T\left(\frac{1}{m}\sum_{i=1}^mx^{(i)}\left(x^{(i)}\right)^T\right)u
\end{align}
$$

利用线性代数中的[关于特征向量的知识](http://nbviewer.jupyter.org/github/zlotus/notes-linear-algebra/blob/master/chapter21.ipynb)以及[关于对称矩阵的知识](http://nbviewer.jupyter.org/github/zlotus/notes-linear-algebra/blob/master/chapter26.ipynb)，我们很容易看出在$\lVert u\rVert_2=1$时最大化上面的式子将会得到对称矩阵$\displaystyle\varSigma=\frac{1}{m}\sum_{i=1}^mx^{(i)}\left(x^{(i)}\right)^T$的主特征向量，该矩阵也正好是关于数据集的协方差矩阵（假设数据的期望为零）。（如果以前没遇到过这种式子，可以尝试使用拉格朗日乘数法，在约束条件$u^Tu=1$下最大化$u^T\varSigma u$。构造拉格朗日算子$\displaystyle\mathcal L(u,\lambda)=u^T\varSigma u-\lambda\left(u^Tu-1\right)$，再对$u$求偏导$\nabla_u\mathcal L(u)=\varSigma u-\lambda u$，置为零后我们最终得到$\varSigma u=\lambda u$，也就是说$u$是$\varSigma$的特征值为$\lambda$时所对应的特征向量。）

总结一下，如果我们希望找到大致符合数据分布的一维子空间的方向$u$，那么就应该选择协方差矩阵$\varSigma$的主特征向量作为这个方向。推广到一般情况，如果我们希望将数据投影到$k$维子空间中（$k\lt n$），我们应该选择$\varSigma$的前$k$个特征向量（即特征值从大到小排列时，前$k$个特征值所对应的特征向量）作为$u_1,\cdots,u_k$。此时$u_i$成为了数据的一组新的正交基。（因为$\varSigma$是对称矩阵，则其特征向量$u_i$将相互正交，或着说总能选出相互正交的特征向量。）

现在，需要把$x^{(i)}$用这组新的基表示出来，我们只需要计算：

$$
y^{(i)}=\begin{bmatrix}u_1^Tx^{(i)}\\u_2^Tx^{(i)}\\\vdots\\u_k^Tx^{(i)}\end{bmatrix}\in\mathbb R^k
$$

与$x^{(i)}\in\mathbb R^n$相比，此时的$y^{(i)}\in\mathbb R^k$，也就是将原来的样本（$n$维）$x^{(i)}$近似表达为更低维度（$k$维）的$y^{(i)}$。因此PCA也被看做是一种降维算法，而向量$u_1,\cdots,u_k$也称作数据的前$k$个主成分。

**注意：**尽管我们只演示了$k=1$的情况，使用特征向量的性质就可以知道，在所以可能的基$u_1,\cdots,u_k$的选择中，我们选择的这组基最大化了$\displaystyle\sum_{i=1}^m\left\lVert y^{(i)}\right\rVert_2^2$。因此，这组基尽可能多的保留了原数据集中的信息。

在[问题集4](http://cs229.stanford.edu/materials/ps4.pdf)中，我们可以看到PCA的另一种推导，即选择一组能够最小化投影时产生的误差（即原数据到投影点之间的距离，图中×到对应黑点之间的距离）之和的基，这种方法同样可以得到由$k$个数据集的主成分张成的$k$维子空间。

PCA算法有很多应用，我们用几个例子来结束关于PCA的讨论。

* 首先是在数据可视化领域用来压缩数据维度，将$x^{(i)}$用维度更低的$y^{(i)}$表示，这是一个很明显的应用场景。如果我们将维数很高的数据降为$2$或$3$维，就可以将数据可视化了。比如我们将例子中的汽车相关数据降为$2$维，我们就可以画出图像（图中的一个点就代表一类车），观察哪些车与其他车类似，哪些车可以聚为一类。
* 另一种应用是做数据的预处理，在使用$x^{(i)}$做监督学习算法的输入之前先将它降维。除了减少计算量，数据降维还能降低假设类的复杂度，有助于避免过拟合（比如对于低维度输入空间的线性分类器，其VC维度较小）。
* 最后，在我们关于遥控直升机操纵者的调查中，可以看到PCA算法的降噪作用。在例子中PCA在带有噪音的关于“飞行水平”和“热爱程度”的问卷调查中，对“飞机操纵因果关系”做出了较为准确的估计。在课程中也接触到PCA关于人脸图像的应用，于是有了**特征脸（eigenfaces）**算法：输入样本$x^{(i)}\in\mathbb R^{100\times100}$是一个$10000$维向量，这是一张$100\times100$的脸部照片，向量的每一个分量都是照片中的一个像素的亮度值。使用PCA算法后，我们可以使用维度很低的$y^{(i)}$来表示$x^{(i)}$，并且希望我们找到的主成分保留了能够表达面部真实特征的有趣的、有规则的信息，并剔除了因为细微的照明变化、照片状况等因素导致的噪音。之后我们会计算降维后照片$i$与$j$之间的距离$\left\lVert y^{(i)}-y^{(j)}\right\rVert_2$。在实际中的应用结果表明，这是一个很优秀的面部匹配、检索算法。