
# 第四讲：牛顿法、一般线性模型

回到上一讲介绍的逻辑回归：在逻辑回归中，我们用S型函数作为$g(z)$，使用梯度上升的更新规则求参数$\theta$的最大似然估计。下面来介绍另一种方法：

## 7. 牛顿法

我们先回忆一下牛顿法找函数值零点：设函数$f:\ \mathbb{R}\to\mathbb{R}$，我们希望找到使$f(\theta)=0$的$\theta\in\mathbb{R}$值，牛顿法提供下面的更新规则：

$$\theta:=\theta-\frac{f(\theta)}{f'(\theta)}$$

该算法的大概流程为：先做对$\theta$做一次猜测$\theta=\theta_1$，然后过$f(\theta_1)$做$f$的切线，切线交横轴于点$\theta_2$（也就是切线函数值的零点），此时更新$\theta=\theta_2$，再过$f(\theta_2)$做$f$的切线，这样$\theta$就可以逼近使函数值为零的$\theta$取值，下面是牛顿法图解：

<img src="./resource/chapter04_image01.png" width="800" alt="" align=center />

在最左的图中为函数$f$的图像，我们希望找到零$f(\theta)=0$的点，从图中看这个点大约取在$1.3$附近。假设我们初始化算法时猜测$\theta^{(0)}=4.5$，牛顿法会计算出函数$f$在$(4.5,\ f(4.5))$点的切线并解出切线的零点（即切线与$y=0$的焦点，如中图所示）。这就是第二次猜测的起点，在图中$\theta^{(1)}$大约在$2.8$附近。而最右侧的图像显示了再一次迭代的结果，这使得$\theta^{(2)}$取到了$1.8$附近。在几轮迭代之后，$\theta$会快速向$1.3$附近收敛。

根据导数（切线斜率）的定义有$f'\left(\theta^{(0)}\right)=\frac{f\left(\theta^{(0)}\right)}{\Delta}$，于是$\Delta=\frac{f\left(\theta^{(0)}\right)}{f'\left(\theta^{(0)}\right)}$，所以$\theta^{(1)}=\theta^{(0)}-\Delta=\theta^{(0)}-\frac{f\left(\theta^{(0)}\right)}{f'\left(\theta^{(0)}\right)}$。则对于一般情况有$\theta^{(t+1)}=\theta^{(t)}-\frac{f\left(\theta^{(t)}\right)}{f'\left(\theta^{(t)}\right)}$。

牛顿法给了我们一个求$f(\theta)=0$的方法，我们也可以利用它来求函数$\mathscr{l}(\theta)$的最大值。函数的最大值点也就是$\mathscr{l}(\theta)$一阶倒数为零的点，所以只需要替换上面方法中的$f(\theta)=\mathscr{l}'(\theta)$即可，我们可以用牛顿方法，按照下面的更新规则求出函数最大值：

$$\theta:=\theta-\frac{\mathscr{l}'(\theta)}{\mathscr{l}''(\theta)}$$

上一讲中，我们在推导逻辑回归的最大值时，使用向量记法的$\theta$，我们将这一标记法应用到牛顿法中，在这种多维条件下一般化的牛顿法（也称作Newton-Raphson方法）为：

$$\theta:=\theta-H^{-1}\nabla_\theta\mathscr{l}(\theta)$$

* 式中的$\nabla_\theta\mathscr{l}(\theta)$仍然表示$\mathscr{l}(\theta)$关于向量$\theta$的每一个分量$\theta_i$的偏导数；
* $H$是一个$n$阶方阵（如果算上截距项应该是$n+1$阶方阵），称为**海森矩阵（Hessian matrix）**，为了便于记忆可以当做是一阶导数（向量）除以二阶导数（乘以方阵的逆），方阵的定义为：

$$H_{ij}=\frac{\partial^2\mathscr{l}(\theta)}{\partial\theta_i\partial\theta_j}$$

相比于（批量）梯度下降法，牛顿法收敛速度通常更快，且逼近最小值时需要的迭代次数更少（牛顿收敛是一个二阶收敛（quadratic convergence））。然而牛顿法单次迭代的代价通常比梯度下降法更大，因为它需要计算整个$n$阶方阵再求逆。通常情况下，在$n$不是很大的条件下，牛顿法总体上更快。当我们使用牛顿法求逻辑回归中对数化似然函数$\mathscr{l}(\theta)$的最大值时，这个算法也被称作**Fisher scoring**。

# 第三部分：一般线性模型（Generalized Linear Models）

到目前为止，我们了解了回归问题、分类问题：在回归问题中我们设$y\mid x;\theta\sim\mathcal{N}\left(\mu,\sigma^2\right)$，而在分类问题中我们设$y\mid x;\theta\sim\mathrm{Bernoulli}(\phi)$，对关于$x$和$\theta$的函数给出合理的$\mu$和$\phi$。我们会在这一节看到，这两种假设都是一族算法模型中的特例，这一族算法模型就是我们曾经提到的一般线性模型（GLMs）。我们还将了解别的GLM家族中的模型如何推导、应用于其他分类、回归问题中。

## 8. 指数族（exponential family）

我们从定义指数分布族开始介绍GLMs，其中属于指数族的分布都可以写为如下形式：

$$p(y;\eta)=b(y)\exp\left(\eta^TT(y)-a(\eta)\right)\tag{1}$$

* 式中的$\eta$称为分布的**自然参数（natural parameter）**或**规范参数（canonical parameter）**；
* $T(y)$称为**充分统计量（sufficient statistic）**（对于我们涉及的分布，通常$T(y)=y$）；
* $a(\eta)$称为**对数配分函数（log partition function）**
* $e^{-a(\eta)}$这个量起到标准化常量的作用，它使得$p(y;\eta)$求和（离散）或对$y$求积分（连续）的结果等于$1$。

如果选定了$T,\ a,\ b$，就能够定义出由$\eta$参数化的一族（或一个集合）分布，之后，我们通过改变$\eta$就能获得这一族分布中不同的分布函数。

我们将展示伯努利分布和高斯分布都是指数分布族中的特例。期望为$\phi$的伯努利分布写作$\mathrm{Bernoulli}(\phi)$，分布$y\in\{0,\ 1\}$上取值，所以$p(y=1;\phi)=p;\ p(y=0;\phi)=1-\phi$，我们改变$\phi$则会得到有不同期望值的伯努利分布。这一类由$\phi$确定的伯努利分布就在指数分布族中，即将$(1)$式中的$T,\ a,\ b$确定下来后，式子就可以化为伯努利分布类型的表达式。

我们将伯努利分布写为：

$$\begin{align}p(y;\phi)&=\phi^y(1-\phi)^{1-y}\\&=\exp(y\log\phi+(1-y)\log(1-\phi))\\&=\exp\left(\left(\log\left(\frac{\phi}{1-\phi}\right)\right)y+log(1-\phi)\right)\end{align}$$

因此，自然参数为$\eta=\log\frac{\phi}{1-\phi}$，有趣的是，如果我们反过来用从$\eta$的表达式中解出$\phi$，则有$\phi=\frac{1}{1+e^{-\eta}}$，这就是逻辑回归中使用的S型函数！当我们用一般线性模型（GLM）的形式表达逻辑回归时，还将再次看到这一现象。为了将伯努利分布完全用指数族分布的形式表达，我们给出其它关于指数分布族的参数：

$$\begin{align}T(y)&=y\\a(\mu)&=-\log(1-\phi)\\&=\log(1+e^{\eta})\\b(y)&=1\end{align}$$

上面的参数表明，伯努利分布在选取适当的参数$T,\ a,\ b$后，就可以写为$(1)$式的形式。

接下来我们考虑高斯分布。回忆由概率模型推导线性回归的过程中，我们发现方差$\sigma^2$的取值并不影响我们最终得到的参数$\theta$和假设函数$h_\theta(x)$，因此我们即使选择任意的$\sigma^2$也不会对分布的表示造成任何影响。为了便于推导，令$\sigma^2=1$。（即使我们将$\sigma^2$以变量的形式留在概率密度函数中，高斯分布也可以表达成指数分布族的形式，此时$\mu\in\mathbb{R}^2$变为一个由$\mu,\ \sigma$表示的向量。为了将带有$\sigma^2$的高斯分布纳入GLM，我们可以给出更一般的指数族定义：

$$p(y;\eta,\tau)=b(a,\tau)\exp\left(\frac{\eta^TT(y)-a(\eta)}{c(\tau)}\right)$$

上式中的$\tau$称为**离散参数（dispersion parameter）**，而高斯分布对应的$c(\tau)=\sigma^2$。不过，对于我们简化的高斯分布，并不需要这种更一般的表达。）

$$\begin{align}p(y;\mu)&=\frac{1}{\sqrt{2\pi}}\exp\left(-\frac{1}{2}(y-\mu)^2\right)\\&=\frac{1}{\sqrt{2\pi}}\exp\left(-\frac{1}{2}y^2\right)\cdot\exp\left(\mu y-\frac{1}{2}\mu^2\right)\end{align}$$

于是，将高斯分布写为指数族形式有：

$$\begin{align}\eta&=\mu\\T(y)&=y\\a(\eta)&=\frac{\mu^2}{2}=\frac{\eta^2}{2}\\b(y)&=\frac{1}{\sqrt{2\pi}}\exp\left(\frac{-y^2}{2}\right)\end{align}$$

指数族当然还有很多别的成员，比如：

* 多项分布（multinomial），我们将在后面介绍，如同伯努利分布对有两个结果的事件建模一样，多项分布对有$K$个结果的事件建模；
* 泊松分布（Poisson），用于计数数据建模，比如样本中放射性元素衰变的书目、某网站访客的数量、商店顾客的数量；
* 伽马分布和指数分布（the gamma and the exponential），用于非负的连续随机变量建模，如处理时间间隔，或是处理在等公交时发生的“下一辆车什么时候到”的问题；
* 贝塔分布和狄利克雷分布（the beta and the Dirichlet），通常用于对分数进行建模，它是对概率分布进行建模的概率分布；
* Wishart分布，这是协方差矩阵的分布。

后面的课程将介绍一种常规方法，来建立对于给定了$x,\theta$的来自上述任意分布的$y$的模型。

## 9. 一般线性模型建模

假如现在需要对我们商店在给定时间内客户数量$y$做出估计，估计的依据来自诸如商店促销活动、近期广告投入、当地天气状况、给定时间是周几等等，将这些作为特征值$x$，我们也知道泊松分布适用于预测来访者数量，那接下来该如何针对问题建立模型呢？幸运的是，泊松分布属于指数分布族，于是我们可以应用一般线性模型（GLM）。在这一节，我们将介绍针对类似问题建立一般线性模型的方法。

一般的，对于想要预测的关于$x$的随机变量$y$的分类或回归问题，我们为了推导出关于问题的一般线性模型，模型需要遵守以下三条关于给定$x$下$y$的条件概率的假设：

1. $y\mid x;\theta\sim\mathrm{ExponentialFamily(\eta)}$，即对于给定的$x,\ \theta$，分布$y$服从某个由$\eta$参数化的指数族分布；
2. 对于给定的$x$，我们的目标是求出对于条件$x$下$T(y)$的期望，即$\mathrm{E}[T(y)\mid x]$。而在大多数情况下$T(y)=y$，所以我们希望学习算法得到的$h(x)=\mathrm{E}[y\mid x]$。（注意，在关于$h_\theta(x)$的选择上，这条假设既适用于逻辑回归又适用于线性回归，比如，$h_\theta(x)=p(y=1\mid x;\theta)=0\cdot p(y=0\mid x;\theta)+1\cdot p(y=1\mid x;\theta)=\mathrm{E}[y\mid x;\theta]$。）
3. 自然参数$\eta$（通常是实数）和输入特征值$x$是线性关系：$\eta=\theta^Tx,\ \eta\in\mathbb{R}$（或者对于向量有$\eta_i=\theta_i^Tx,\ \eta\in\mathbb{R}^*$。）

第三条也许是这些假设中“最欠合理”的了，它看起来更像是构建一般线性模型的”设计决策“，而不是”假设“。这三条假设（设计决策）可以让我们得到一类十分简洁的学习算法，也就是一般线性模型，并且具有良好的特性，比如便于我们学习。此外，所得模型通常对关于$y$的不同类型分布的建模都有良好的效果，比如我们接下来将介绍一般线性模型既能推导出逻辑回归，又能推导出普通最小二乘。

### 9.1 普通最小二乘法

为了证明普通最小二乘是一般线性模型中的特例，需要考虑目标变量$y$（在一般线性模型术语中也称作**响应变量（response variable）**）是连续的，我们使用高斯分布$\mathcal{N}\left(\mu,\ \sigma^2\right)$建立关于给定$x$下$y$的条件概率模型（$\mu$可能与$x$有关）。所以我们令上面的$\mathrm{ExponentialFamily}(\eta)$为高斯分布。在前面推导出的将高斯分布写为指数分布族的公式中，有$\mu=\eta$，所以：

$$\begin{align}h_\theta(x)&=\mathrm{E}[y\mid x;\theta]\\&=\mu\\&=\eta\\&=\theta^Tx\end{align}$$

* 第一个式子来自假设2；
* 第二个式子根据$y\mid x;\theta\sim\mathcal{N}\left(\mu,\sigma^2\right)$，$y$的期望值就是$\mu$；
* 第三个式子来自假设1（结合前推导出的将高斯分布写为指数分布族的公式）；
* 第四个式子来自假设3。

### 9.2 逻辑回归

再来考虑逻辑回归。现在主要看二元分类，也就是$y\in\{0,1\}$，我们很自然的联想到使用伯努利族分布对$y$建模。在前面推导出的将伯努利分布写为指数分布族的公式中，$\phi=\frac{1}{1+e^{-\eta}}$。此外，如果$y\mid x;\theta\sim\mathrm{Bernoulli}(\phi)$，则$\mathrm{E}[y\mid x;\theta]=\phi$。同上面推导普通最小二乘一样：

$$\begin{align}h_\theta(x)&=\mathrm{E}[y\mid x;\theta]\\&=\phi\\&=\frac{1}{1+e^{-\eta}}\\&=\frac{1}{1+e^{-\theta^Tx}}\end{align}$$

于是我们得到了假设函数$h_\theta(x)=\frac{1}{1+e^{-\theta^Tx}}$，这就是上讲中逻辑函数的来源：一旦我们假设对给定$x$的$y$的条件概率服从伯努利分布，就可以推导出伯努利分布的指数分布族形式，进而得到一般线性模型的定义。

再引入一些术语，一个关于自然参数$\eta$的函数$g$给出了分布期望（$g(\eta)=\mathrm{E}[T(y);\eta]$），我们把这个函数称作**正则响应函数（canonical response function）**，其逆$g^{-1}$称作**正则关联函数（canonical link function）**。对于高斯分布，正则响应函数就是$g(\eta)=\eta$；对于伯努利分布就是逻辑函数。（很多其他教材使用$g$正则连接函数，而用$g^{-1}$表示正则响应函数，但我们在这里的表示法沿用以往的机器学习课程，并将在后面的课程中继续使用这种记法。）

### 9.3 Softmax回归

再多看一个一般线性模型的例子：如果在分类问题中，遇到$y$能够取$k$个值（不再限于两个）的情况，即$y\in\{1,2,\cdots,k\}$。继续用垃圾邮件的例子，我们前面将一封邮件通过算法标记为”垃圾邮件“或”正常邮件“，这是一个二元分类问题，现在我们想将其分为三类：”垃圾邮件“、”私人邮件“和”工作邮件“。这个问题中，响应变量仍然是离散值，只不过变成了两个以上的值，我们因此需要使用多项分布来建立模型。

要推导此类问题的一般线性模型，我们首先得将多项分布表达成指数分布族的形式。

为了参数化有$k$个可能取值的多项分布，我们使用$k$个参数$\phi_1,\cdots,\phi_k$，用来表示每个可能得取值发生的概率。然而，这样设置参数可能会有冗余（过度参数化，over-parameterized），也就是参数线性相关（如果知道任意$k-1$个$\phi_i$，都可以使用$\displaystyle\sum_{i=1}^k\phi_i=1$算出最后一个$未知\phi$的值）。所以我们应该设置$k-1$个参数$\phi_1,\cdots,\phi_{k-1}$，则$\phi_i=p(y=i;\phi)$，$p(y=k;\phi)=1-\displaystyle\sum_{i=1}^{k-1}\phi_i$。为了写起来方便，在后面的推导中，我们令$\phi_k=1-\displaystyle\sum_{i=1}^{k-1}\phi_i$，但得清楚这不是一个参数，它是从其他参数中计算出来的。

接下来我们需要定义$T(y)\in\mathbb{R}^{k-1}$（多项分布是少数几个$T(y)\neq y$的概率模型）：

$$T(1)=\begin{bmatrix}1\\0\\0\\\vdots\\0\end{bmatrix},\ T(2)=\begin{bmatrix}0\\1\\0\\\vdots\\0\end{bmatrix},\ T(3)=\begin{bmatrix}0\\0\\1\\\vdots\\0\end{bmatrix},\ \cdots,\ T(k-1)=\begin{bmatrix}0\\0\\0\\\vdots\\1\end{bmatrix},\ T(k)=\begin{bmatrix}0\\0\\0\\\vdots\\0\end{bmatrix}$$

与前面介绍的分布不同的是，这里不再是$T(y)=y$，现在的$T(y)$不再是一个实数，它变成了一个$k-1$维向量，我们使用$(T(y))_i$表示向量$T(y)$的第$i$个分量。

我们再引入一个非常有用的标记：指示函数（indicator function）$1\{\cdot\}$，如果参数为真则返回$1$，反之返回$0$（$1\{True\}=1,\ 1\{False\}=0$）。举个例子$1\{2=3\}=0$而$1\{3=5-2\}=1$。

我们可以利用这个函数将$T(y)$与$y$的关系表示成$(T(y))_i=1\{y=i\}$（则$T(y)=\begin{bmatrix}(T(y))_1\\(T(y))_2\\\vdots\\(T(y))_{k-1}\end{bmatrix}=\begin{bmatrix}1\{y=1\}\\1\{y=2\}\\\vdots\\1\{y=k-1\}\end{bmatrix}$，这样就可以简洁的表示上面的一系列向量了）。更进一步，有$\mathrm{E}[(T(y))_i]=P(y=i)=\phi_i$。

一切准备就绪，可以开始将多项分布化为为指数分布族了：

$$\begin{align}p(y;\phi)&=\phi_1^{1\{y=1\}}\phi_2^{1\{y=2\}}\cdots\phi_{k-1}^{1\{y=k-1\}}\phi_k^{1\{y=k\}}\\&=\phi_1^{1\{y=1\}}\phi_2^{1\{y=2\}}\cdots\phi_{k-1}^{1\{y=k-1\}}\phi_k^{1-\sum_{i=1}^{k-1}1\{y=i\}}\\&=\phi_1^{(T(y))_1}\phi_2^{(T(y))_2}\cdots\phi_{k-1}^{(T(y))_{k-1}}\phi_k^{1-\sum_{i=1}^{k-1}(T(y))_i}\\&=\exp\left((T(y))_1\log(\phi_1)+(T(y))_2\log(\phi_2)+\cdots+(T(y))_{k-1}\log(\phi_{k-1})+\left(1-\sum_{i=1}^{k-1}(T(y))_i\right)\log(\phi_k)\right)\\&=\exp\left((T(y))_1\log\left(\frac{\phi_1}{\phi_k}\right)+(T(y))_2\log\left(\frac{\phi_2}{\phi_k}\right)+\cdots+(T(y))_{k-1}\log\left(\frac{\phi_{k-1}}{\phi_k}\right)+\log(\phi_k)\right)\\&=b(y)\exp\left(\eta^TT(y)-a(\eta)\right)\end{align}$$

对于此式有（注意这里的$\phi_k=1-\displaystyle\sum_{i=1}^{k-1}\phi_i$，它并不是一个参数）：

$$\begin{align}\eta^T&=\begin{bmatrix}\log\frac{\phi_1}{\phi_k}&\log\frac{\phi_2}{\phi_k}&\cdots&\log\frac{\phi_{k-1}}{\phi_k}\end{bmatrix}\\a(\eta)&=-\log(\phi_k)\\b(y)&=1\end{align}$$

于是我们将多项分布写成了指数分布族的形式。

正则连接函数为（参数$\eta$关于期望$\phi$的函数）：

$$\eta_i=\log\frac{\phi_i}{\phi_k}\quad (i=1,\cdots,k-1)$$

为了表示方便，我们令$\eta_k=\log\frac{\phi_k}{\phi_k}=0$，求正则响应函数（即正则连接函数的逆，期望$\phi$关于参数$\eta$的函数）：

$$\begin{align}e^{\eta_i}&=\frac{\phi_i}{\phi_k}\\\phi_ke^{\eta_i}&=\phi_i\\\phi_k\sum_{i=1}^ke^{\eta_i}&=\sum_{i=1}^k\phi_i=1\end{align}$$

第三个式子表明$\phi_k=\frac{1}{\sum_{i=1}^ke^{\eta_i}}$，代回第二个式子得到正则响应函数：

$$\phi_i=\frac{e^{\eta_i}}{\sum_{i=1}^ke^{\eta_i}}\quad (i=1,\cdots,k-1)$$

这个函数是由$\eta$到$\phi$的映射，也叫作**softmax**函数。

接下来需要使用假设3，也就是$\eta_i$是输入变量$x$的线性函数，这次我们要使用括号里关于向量$\eta_i$的定义：$\eta_i=\theta_i^Tx,\ \eta\in\mathbb{R}^{k-1},\ i=1,\cdots,k-1$，这里$\theta$仍旧是特征值$x$的系数，有$\theta_1,\cdots,\theta_{k-1}\in\mathbb{R}^{n+1}$（$n$是训练样本所取特征值的个数）。为了方便公式书写，我们依旧令$\theta_k=0$，则多出来一项$\eta_k=\theta_k^Tx=0$，于是在$x$条件下$y$的概率可以表示为：

$$\begin{align}p(y=i\mid x;\theta)&=\phi_i\\&=\frac{e^{\eta_i}}{\sum_{i=1}^ke^{\eta_j}}\\&=\frac{e^{\theta_i^Tx}}{\sum_{j=1}^ke^{\theta_j^Tx}}\tag{2}\end{align}$$

这就是在$y\in\{1,\cdots,k\}$取值下的分类问题模型，也叫作**softmax回归**，是一般化的逻辑回归。

此时的假设函数为：

$$\begin{align}h_\theta(x)&=\mathrm{E}[T(y)\mid x;\theta]\\&=\mathrm{E}\left[\begin{array}{c|c}1\{y=1\}&\\1\{y=2\}&\\\vdots&x;\theta\\1\{y=k-1\}&\end{array}\right]\\&=\begin{bmatrix}\phi_1\\\phi_2\\\vdots\\\phi_{k-1}\end{bmatrix}\\&=\begin{bmatrix}\frac{\exp\left(\theta_1^Tx\right)}{\sum_{j=1}^k\exp\left(\theta_j^Tx\right)}\\\frac{\exp\left(\theta_2^Tx\right)}{\sum_{j=1}^k\exp\left(\theta_j^Tx\right)}\\\vdots\\\frac{\exp\left(\theta_{k-1}^Tx\right)}{\sum_{j=1}^k\exp\left(\theta_j^Tx\right)}\end{bmatrix}\end{align}$$

可以看出，假设函数会对$p(y=i\mid x;\theta)$的每一个$i=1,\cdots,k$做出概率估计（尽管我们仅把$h_\theta(x)$定义在$k-1$维，但显然第$k$个情况的概率估计$p(y=k\mid x;\theta)$可以从$1-\sum_{i=1}^{k-1}\phi_i$算出）。

最后一步就是求拟合参数了。与普通最小二乘、逻辑回归中的推导方法相似，如果训练集中有$m$个训练样本$\left\{(x^{(i)},y^{(i)});i=1,\cdots,m\right\}$，想通过学习算法得到$\theta_i$，则需要同以前一样，先写出似然函数：

$$\begin{align}L(\theta)&=\prod_{i=1}^mp\left(y^{(i)}\mid x^{(i)};\theta\right)\\&=\prod_{i=1}^m\left(\phi_1^{1\left\{y^{(i)}=1\right\}}\phi_2^{1\left\{y^{(i)}=2\right\}}\cdots\phi_k^{1\left\{y^{(i)}=k\right\}}\right)\\&=\prod_{i=1}^m\left(\prod_{l=1}^k\left(\frac{\exp\left(\theta_l^Tx^{(i)}\right)}{\sum_{j=1}^k\exp\left(\theta_j^Tx^{(j)}\right)}\right)^{1\left\{y^{(i)}=l\right\}}\right)\end{align}$$

再对似然函数取对数：

$$\begin{align}\mathscr{l}(\theta)&=\log L(\theta)\\&=\sum_{i=1}^m\log\prod_{l=1}^k\left(\frac{\exp\left(\theta_l^Tx^{(i)}\right)}{\sum_{j=1}^k\exp\left(\theta_j^Tx^{(j)}\right)}\right)^{1\left\{y^{(i)}=l\right\}}\end{align}$$

在上面的推导中我们使用了$(2)$式给出的$p(y=i\mid x;\theta)$的定义，剩下的就是计算$\mathscr{l}(\theta)$中参数$\theta$的最大似然估计了，计算过程可以使用前面介绍的梯度上升或牛顿法。（如果取$n$个特征值，则向量$x\in\mathbb{R}^{n+1}$（含截距项$x_0=1$），有$\theta_j\in\mathbb{R}^{n+1},\ j=1,\cdots,k-1$（也就是对每一个可能的$y_j$都有一套$\theta_j$拟合，线性无关的共$k-1$个），如果愿意的话可以把$\theta_j,\ i=1,\cdots,k-1$组成一个矩阵，即系数矩阵$\theta\in\mathbb{R}^{(n+1)\times(k-1)}$。）

一般线性模型建模过程可以概括为：

1. 根据训练集$x^{(i)},y^{(i)}$选择概率分布模型，参数为$\phi$；
2. 将该分布写为指数分布族的形式，参数为$\eta$；
3. 可以得到正则响应函数$g(\eta)=\mathrm{E}[T(y);\eta]$；
4. 将$\eta=\theta^Tx$带入正则响应函数得到假设函数$h_\theta(x)=g(\theta^Tx)$；
5. 根据模型的概率解释得到似然函数$L(\theta)=p(y^{(i)}\mid x^{(i)};\theta)$（根据假设函数得到）；
6. 取合适的$\theta$使似然函数最大化。