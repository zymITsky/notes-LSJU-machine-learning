
# 第七讲：最优间隔分类器、拉格朗日对偶、支持向量机

简单的回顾上一讲的内容：

* 我们定义了新的假设函数：

    $$\begin{align}h_{w,b}&=g\left(w^Tx+b\right)\\g(z)&=\begin{cases}1 &z\gt 0\\-1 &z\lt 0\end{cases}\\y&=\{-1,1\}\end{align}$$

* 给出了函数间隔的定义：

    $$\hat{\gamma}^{(i)}=y^{(i)}\left(w^Tx^{(i)}+b\right)$$

* 以及几何间隔的定义：

    $$\gamma^{(i)}=y^{(i)}\left(\left(\frac{w}{\lVert w\rVert}\right)^Tx^{(i)}+\frac{b}{\lVert w\rVert}\right)$$

* 了解了函数间隔与几何间隔的关系：

    $$\gamma=\frac{\hat\gamma}{\lVert w\rVert}$$

* 同时也了解了其几何意义：由$w^Tx+b=0$确定的分类超平面将正负样本分隔开，同时使超平面与样本间的最小间隔尽可能大。我们也发现，如果按比例缩放参数$(w,b)$，并不会对假设结果造成任何影响，因为给参数同时乘以一个系数并不会影响超平面的位置。

接着上一讲，我们继续介绍支持向量机的相关知识。

## 4. 最优间隔分类器（optimal margin classifier）

通过上一讲的介绍，对于给定的训练集，我们会自然的想要找到一个能够使（几何）间隔达到最大的判定边界，因为这样的判定边界能够得到高度可信的预测集，同时也是对训练集很好的拟合。这也将帮助我们得到一个将正负样本分隔（几何间隔）开的分类器。

目前为止，我们都假设训练集是线性可分的，即我们一定可以找到一个能够将正负样本分开的分类超平面。那么，我们怎样找到将几何间隔最大化的分类超平面呢？讨论下面的优化问题：

$$\begin{align}\displaystyle\operatorname*{max}_{w,b}&\quad\gamma\\\mathrm{s.t.}&\quad y^{(i)}\left(w^Tx^{(i)}+b\right)\geq\gamma,\quad i=1,\cdots,m\\&\quad\lVert w\rVert=1\end{align}$$

上式表示，在满足每一个训练样本的函数间隔都不小于$\gamma$的情况下，使用$(w,b)$构造$\gamma$的能够取到的最大值，同时有限制条件$\lVert w\rVert=1$保证函数间隔等于几何间隔。通过求得的$(w,b)$就可以计算出该训练集的最大几何间隔。（注：$\mathrm{s.t.}$是“subject to”的缩写，意为“受限于”。）

上面的优化问题有个棘手的限制条件$\lVert w\rVert=1$（这是一个糟糕的非凸性约束，即参数$w$可能位于一个单位圆环/球面上），这显然不是可以提交给标准的凸优化软件（如梯度下降法、牛顿法等）去解决的问题。所以，我们将上面的问题变形为另一种的形式：

$$\begin{align}\displaystyle\operatorname*{max}_{w,b}&\quad\frac{\hat\gamma}{\lVert w\rVert}\\\mathrm{s.t.}&\quad y^{(i)}\left(w^Tx^{(i)}+b\right)\geq\hat\gamma,\quad i=1,\cdots,m\end{align}$$

也就是我们选择最大化几何间隔$\frac{\hat\gamma}{\lVert w\rVert}$，同时满足所有样本的函数间隔不小于$\hat\gamma$的条件。因为函数间隔和几何间隔的关系为$\gamma=\frac{\hat\gamma}{\lVert w\rVert}$，而又舍弃了限制条件$\lVert w\rVert=1$，所以这次我们觉得应该可以求出最大值了。不过值得注意的是，这次最大化的对象$\frac{\hat\gamma}{\lVert w\rVert}$是非凸的，而我们同样没有现成的凸优化软件来解决此类问题。

于是，我们发现在第一个优化问题中，存在非凸性的限制条件，而在第二个优化问题中，存在非凸性的优化目标，所以我们并不能保证软件可以找到全局最小值。（要带上限制条件$\lVert w\rVert=1$是因为我们的优化想要度量的是几何间隔）。

回想前一讲，我们知道，按比例缩放参数$(w,b)$对假设结果没有任何影响，我们现在可以利用这一点。我们现在来引入限制条件：对于给定的训练集，以$(w,b)$为参数的函数间隔必须为$1$：

$$\hat\gamma=1$$

给参数$w,b$乘以缩放常量会使得函数间隔缩放同样的倍数，则可以通过缩放$w,b$来满足上面的限制条件。将这一限制条件加入上面的假设中，于是问题变为最大化$\frac{\hat\gamma}{\lVert w\rVert}=\frac{1}{\lVert w\rVert}$，也就是相当于最小化$\lVert w\rVert^2$。现在的优化问题变为：

$$\begin{align}\displaystyle\operatorname*{min}_{w,b}&\quad\frac{1}{2}\lVert w\rVert^2\\\mathrm{s.t.}&\quad y^{(i)}\left(w^Tx^{(i)}+b\right)\geq 1,\quad i=1,\cdots,m\end{align}$$

这样，我们就把原来棘手的问题变为了可以用软件高效解决的问题。上式是一个带有线性约束的凸二次型问题，解该式即可得到**最优间隔分类器（optimal margin classifier）**。这一类优化问题可以直接使用一些现成的商用二次优化程序（QP: quadratic programming）解决。

到这里，问题基本解决，我们暂停一下支持向量机的讲解，先了解一下拉格朗日对偶。它将引出我们优化问题的对偶形式，通过使用核方法，将保证在我们将最优间隔分类器问题推广到极高维度空间时算法的高效求解。对偶形式也将帮助我们推导出一个用以解决上面优化问题的高效算法，其求解速度比常见的QP软件快很多。

## 5. 拉格朗日对偶（Lagrange duality）

我们先放一放支持向量机和最大间隔分类器，来聊一聊带有约束条件的优化问题。考虑下面的问题：

$$\begin{align}\displaystyle\operatorname*{min}_{w}&\quad f(w)\\\mathrm{s.t.}&\quad h_i(w)=0,\quad i=1,\cdots,l\end{align}$$

回忆在高等数学中学到的拉格朗日乘数法（[中文](https://zh.wikipedia.org/zh/%E6%8B%89%E6%A0%BC%E6%9C%97%E6%97%A5%E4%B9%98%E6%95%B0)，[英文](https://en.wikipedia.org/wiki/Lagrange_multiplier)），我们定义**拉格朗日算子（Lagrangian）**：

$$\mathcal{L}(w,\beta)=f(w)+\sum_{i=1}^l\beta_ih_i(w)$$

此处的系数$\beta$称作**拉格朗日乘数（Lagrange multiplier）**，然后将$\mathcal{L}$的偏导数置为零：

$$\begin{align}\frac{\partial\mathcal{L}}{\partial w_i}&=0\\\frac{\partial\mathcal{L}}{\partial\beta_i}&=0\end{align}$$

最后解出$w,\beta$即可。直接参考拉格朗日乘数法的例题（[中文](https://zh.wikipedia.org/wiki/%E6%8B%89%E6%A0%BC%E6%9C%97%E6%97%A5%E4%B9%98%E6%95%B0#.E4.BE.8B.E5.AD.90)，[英文](https://en.wikipedia.org/wiki/Lagrange_multiplier#Examples)）更加直观。

在这一节我们会一般化这种带有约束条件的优化问题，也就是不限于等式约束条件，后面会加入不等式约束条件。因为篇幅限制，我们不会在这里直接推导拉格朗日对偶理论（感兴趣可以参考R.T. Rockarfeller (1970), Convex Analysis, Princeton University Press.），仅在这里给出主要思路和结果，进而在应用在最优间隔分类器优化问题中。

下面介绍**原始优化问题（primal optimization problem）**：

$$\begin{align}\displaystyle\operatorname*{min}_{w}&\quad f(w)\\\mathrm{s.t.}&\quad g_i(w)\leq 0,\quad i=1,\cdots ,k\\&\quad h_i(w)=0,\quad i=1,\cdots,l\end{align}$$

定义**广义拉格朗日算子（generalized Lagrangian）**：

$$\mathcal{L}(w,\alpha,\beta)=f(w)+\sum_{i=1}^k\alpha_ig_i(w)+\sum_{i=1}^l\beta_ih_i(w)$$

这里的$\alpha,\beta$是拉格朗日乘数，考虑下面的量：

$$\theta_{\mathcal{P}}(w)=\operatorname*{max}_{\alpha,\beta:\alpha_i\geq 0}\mathcal{L}(w,\alpha,\beta)$$

其中下标$\mathcal{P}$代表"primal"。对于一些$w$，如果$w$违反了任何原始约束条件（如$g_i(w)\gt 0$或$h_i(w)\neq 0$），则可以证明：

$$\begin{align}\theta_{\mathcal{P}}(w)&=\operatorname*{max}_{\alpha,\beta:\alpha_i\geq 0}f(w)+\sum_{i=1}^k\alpha_ig_i(w)+\sum_{i=1}^l\beta_ih_i(w)\tag{1}\\&=\infty\tag{2}\end{align}$$

（如果某个$g_i(w)\gt 0$，则拉格朗日算子中该项相应的拉格朗日乘数$\alpha_i$只需取无穷大就可以使整个式子最大化；类似的，如果某个$h_i(w)\neq 0$，则拉格朗日算子中该项相应的拉格朗日乘数$\beta_i$只需取无穷大也可以使整个式子最大化。）

相反，如果$w$在约束条件内，则有$\theta_{\mathcal{P}}=f(w)$，于是有：

$$\theta_{\mathcal{P}}=\begin{cases}f(w)&\textrm{if }w\textrm{ satisfies primal constraints}\\\infty&\textrm{otherwise}\end{cases}$$

（也就是说，对于满足约束条件的$w$，要使得拉格朗日算子最大化，需要拉格朗日乘数项之和为零。）

则对于满足原始约束的$w$来说，$\theta_{\mathcal{P}}$与原始优化问题中的目标函数相同；对于违反原始约束的$w$来说，$\theta_{\mathcal{P}}$为正无穷。因此，如果考虑最小化：

$$\operatorname*{min}_{w}\theta_{\mathcal{P}}(w)=\operatorname*{min}_{w}\operatorname*{max}_{\alpha,\beta:\alpha_i\geq 0}\mathcal{L}(w,\alpha,\beta)$$

我们发现，这与原始优化问题是一样的（也有同样的解）。为了后面使用方便，我们定义优化目标的最优值为$p^*=\displaystyle\operatorname*{min}_{w}\theta_{\mathcal{P}}(w)$，称为原始问题的**最优值（optimal value）**。

现在，我们来看一个略微不同的问题，定义关于拉格朗日乘数的函数：

$$\theta_{\mathcal{D}}(\alpha,\beta)=\operatorname*{min}_{w}\mathcal{L}(w,\alpha,\beta)$$

其中下标$\mathcal{D}$代表“dual”。留意到在$\theta_{\mathcal{P}}$的定义中，我们优化（最大化）关于$\alpha,\beta$的函数；而在这里，我们最小化关于$w$的函数。

引入**对偶优化问题（dual optimization problem）**：

$$\operatorname*{max}_{\alpha,\beta:\alpha_i\geq 0}\theta_{\mathcal{D}}(\alpha,\beta)=\operatorname*{max}_{\alpha,\beta:\alpha_i\geq 0}\operatorname*{min}_{w}\mathcal{L}(w,\alpha,\beta)$$

这个式子除了“max”和“min”的顺序发生改变以外，其余的同前面的原始问题一样。同样的，定义对偶优化问题的最优值为$d^*=\displaystyle\operatorname*{max}_{\alpha,\beta:\alpha_i\geq 0}\theta_{\mathcal{D}}(\alpha,\beta)$。

原始问题与对偶问题的关系可以用下面的式子简单表示：

$$d^*=\operatorname*{max}_{\alpha,\beta:\alpha_i\geq 0}\operatorname*{min}_{w}\mathcal{L}(w,\alpha,\beta)\leq p^*=\operatorname*{min}_{w}\operatorname*{max}_{\alpha,\beta:\alpha_i\geq 0}\mathcal{L}(w,\alpha,\beta)$$

（一个普遍事实：“min max”某函数总是大于等于“max min”某函数，比如$\displaystyle\operatorname*{max}_{y\in\{0,1\}}\underbrace{\left(\displaystyle\operatorname*{min}_{x\in\{0,1\}}1\{x=y\}\right)}_{0}\leq\displaystyle\operatorname*{min}_{x\in\{0,1\}}\underbrace{\left(\displaystyle\operatorname*{max}_{y\in\{0,1\}}1\{x=y\}\right)}_{1}$）在某些特定情况下，能够得到：

$$d^*=p^*$$

也就是说，我们可以通过解对偶优化问题来得到原始优化问题的最优值（这么做的原因是，对偶问题通常更加简单，而且与原始问题相比，对偶问题具有更多有用的性质，稍后我们会在最优间隔分类器即支持向量机问题中见到），接下来看在什么情况下此式成立。

假设$f$和$g_i$是凸函数（对于$f$的海森矩阵有，当且仅当海森矩阵半正定时，$f$是凸函数。举个例子：$f(w)=w^Tw$是凸的，类似的，所有线性（和仿射）函数也是凸的。另外，即使$f$不可微，它也可以是凸的，不过现在我们不需要这种更一般化的关于凸性的定义），$h_i$是仿射函数（[中文](https://zh.wikipedia.org/wiki/%E4%BB%BF%E5%B0%84%E5%8F%98%E6%8D%A2)，[英文](https://en.wikipedia.org/wiki/Affine_transformation)）（仿射函数/变换是指存在$a_i,b_i$使得$h_i(w)=a_i^Tw+b_i$，而“仿射变换”是指线性变换后加上截距项$b_i$使整体平移，即线性变换是固定原点的，而仿射变换是可以平移的）。进一步假设$g_i$是严格可用的，即对于所有$i$存在$w$能够使$g_i(w)\lt 0$。

在上述假设条件下，附加条件$w^*,\alpha^*,\beta^*$一定存在（$w^*$是原始问题的解，$\alpha^*,\beta^*$是对偶问题的解），再附加条件$p^*=d^*=\mathcal{L}(w^*,\alpha^*,\beta^*)$，再附加条件$w^*,\alpha^*,\beta^*$满足**KKT条件（Karush-Kuhn-Tucker conditions）**：

$$\begin{align}\frac{\partial}{\partial w_i}\mathcal{L}(w^*,\alpha^*,\beta^*)&=0,\quad i=1,\cdots,n\tag{3}\\\frac{\partial}{\partial \beta_i}\mathcal{L}(w^*,\alpha^*,\beta^*)&=0,\quad i=1,\cdots,l\tag{4}\\\alpha_i^*g_i(w^*)&=0,\quad i=1,\cdots,k\tag{5}\\g_i(w^*)&\leq0,\quad i=1,\cdots,k\tag{6}\\\alpha_i^*&\geq0,\quad i=1,\cdots,k\tag{7}\end{align}$$

如果存在满足KKT条件的$w^*,\alpha^*,\beta^*$，则原始问题与对偶问题一定有解。$(5)$式又称为**KKT对偶互补条件（KKT dual complementarity condition）**，这个条件表明如果$a_i^*\gt0$则$g_i(w^*)=0$（即约束条件$g_i(w^*)\leq0$“激活”，成为一个**活动约束（active constraint）**并处于取等号的状态）。在后面的课程中，我们将通过这个条件知道支持向量机只有一小部分“支持向量”。当讲到序列最小优化算法（SMO）时，KKT对偶互补条件也会给我们一个验证其收敛特征的方法。

## 6. 最优间隔分类器（续）

在上一节，我们引入了原始优化问题，用以求解最优间隔分类器：

$$\begin{align}\displaystyle\operatorname*{min}_{w,b}&\quad\frac{1}{2}\lVert w\rVert^2\\\mathrm{s.t.}&\quad y^{(i)}\left(w^Tx^{(i)}+b\right)\geq 1,\quad i=1,\cdots,m\end{align}$$

我们可以将约束条件写为：

$$g_i(w,b)=-y^{(i)}\left(w^Tx^{(i)}+b\right)+1\leq0$$

对于每个训练样本都有这样一个约束条件。从对偶互补约束条件可知，当$\alpha_i\gt0\implies g_i(w,b)=0$（此时是一个活动约束）$\iff$样本$(x^{(i)},y^{(i)})$的函数间隔为$1$。也就是说，如果该约束是一个活动约束，那么它实际上是将一个不等式条件变为等式条件，这意味着第$i$个训练样本的函数间隔一定等于$1$。考虑下图，实线表示具有最大间隔的分类超平面：

<img src="./resource/chapter07_image01.png" width="400" alt="" align=center />

图中最靠近判定边界（即$w^Tx+b=0$）的点就是具有最小间隔的点，在上图中共有三个这样的点（一负两正），在平行于判别边界的虚线上。于是，一共有三个$\alpha_i$在我们解优化问题的过程中不为零（也就是虚线上的三个点，只有这三个点的拉格朗日乘数不为零，也只有这三个样本的函数间隔等于$1$，其余样本的函数间隔都严格大于$1$。再多说一点，有时会有$g_i,\alpha_i$都等于零的情况，但通常$g_i=0$时$\alpha_i$是非零的，所以那些函数间隔为$1$的样本就是那些$\alpha_i$不等于零的样本），这三个样本也称为这个问题的**支持向量（support vectors）**。支持向量的数量通常都比训练集样本总量少很多。

随着我们对对偶问题的深入理解，其中的关键思想就是尝试将算法用输入特征空间中点的向量内积$\left\langle x^{(i)},x^{(j)}\right\rangle$的形式表达出来（可以看做$\left(x^{(i)}\right)^Tx^{(j)}$）。实际上当我们使用核方法时，这种表达法将成为算法的关键。

为优化问题构造拉格朗日算子：

$$\mathcal{L}(w,b,\alpha)=\frac{1}{2}\lVert w\rVert^2-\sum_{i=1}^m\alpha_i\left[y^{(i)}\left(w^Tx^{(i)}+b\right)-1\right]\tag{8}$$

式中只有拉格朗日乘数$\alpha_i$而没有$\beta_i$，因为此问题中只含有不等式约束条件。

我们需要找出此问题的对偶形式。要得到对偶问题的$\theta_{\mathcal{D}}$，首先需要最小化关于$w,b$（将$\alpha$作为常量）的函数$\mathcal{L}(w,b,\alpha)$，也就是对$\mathcal{L}$分别求关于$w,b$的偏导，并将偏导数置为零：

$$\nabla_w\mathcal{L}(w,b,\alpha)=w-\sum_{i=1}^m\alpha_iy^{(i)}x^{(i)}=0$$

于是得到$w$：

$$w=\sum_{i=1}^m\alpha_iy^{(i)}x^{(i)}\tag{9}$$

可以看出$w$实际上是由$\alpha_i$设置权重后输入特征值向量的线性组合。继续对$b$求偏导：

$$\frac{\partial}{\partial b}\mathcal{L}(w,b,\alpha)=\sum_{i=1}^m\alpha_iy^{(i)}=0\tag{10}$$

现在将$(9)$式代入$(8)$式并化简，有：

$$\mathcal{L}(w,b,\alpha)=\sum_{i=1}^m\alpha_i-\frac{1}{2}\sum_{i,j=1}^m\alpha_i\alpha_jy^{(i)}y^{(j)}\left(x^{(i)}\right)^Tx^{(j)}-b\sum_{i=1}^m\alpha_iy^{(i)}$$

根据$(10)$式，最后一项应为$0$，于是得到：

$$\mathcal{L}(w,b,\alpha)=\sum_{i=1}^m\alpha_i-\frac{1}{2}\sum_{i,j=1}^m\alpha_i\alpha_jy^{(i)}y^{(j)}\left(x^{(i)}\right)^Tx^{(j)}$$

这个式子是我们通过求函数$\mathcal{L}$关于$w,b$的函数最小值得到的，再加上约束条件$\alpha_i\geq0$（始终存在的约束）和$(10)$式，最终得到下面的对偶优化问题（此时将$\mathcal{L}$看做关于$\alpha$的函数）：

$$\begin{align}\displaystyle\operatorname*{max}_{\alpha}&\quad W(\alpha)=\sum_{i=1}^m\alpha_i-\frac{1}{2}\sum_{i,j=1}^m\alpha_i\alpha_jy^{(i)}y^{(j)}\left\langle x^{(i)},x^{(j)}\right\rangle\\\mathrm{s.t.}&\quad \alpha_i\geq 0,\quad i=1,\cdots,m\\&\quad\sum_{i=1}^m\alpha_iy^{(i)}=0\end{align}$$

（在这里简单的解释一下第二个约束条件，即拉格朗日算子对$b$求偏导的结果。如果$\displaystyle\sum_{i=1}^m\alpha_iy^{(i)}\neq0$，则$\theta_{\mathcal{D}}(\alpha)=-\infty$。换句话说拉格朗日算子是参数$b$的线性函数。所以如果我们的目标是$\displaystyle\operatorname*{max}_{\alpha\geq0}\theta_{\mathcal{D}}(\alpha)$，那么就应该选择使得$\displaystyle\sum_{i=1}^m\alpha_iy^{(i)}=0$的$\alpha$，因为当$\displaystyle\sum_{i=1}^m\alpha_iy^{(i)}=0$时$\theta_{\mathcal{D}}(\alpha)=W(\alpha)$。）

易证$p^*=d^*$及KKT条件（(3)-(7)式）在此优化问题中成立。那么我们就可以通过计算对偶问题来得到原始问题的解。上面的对偶问题是一个求关于$\alpha_i$的函数的最大值的问题。我们稍后再讨论对偶问题的特定解法，先假设我们能够解出最优值（即找到在约束条件下使$W(\alpha)$最大化的$\alpha$的取值），那么就可以使用$(9)$式找到最优值$w$（$w$是一个关于$\alpha$的函数）。考虑原始问题，有了$w^*$则可以直接求出截距项$b$，因为此时超平面的法向量已经确定，我们只需要在满足该法向量的超平面中“固定”适当的截距，使得超平面到正负样本的距离相等即可（即找到图中两虚线之间的实现）。我们将$a,w$带入原始问题求中解$b$：

$$b^*=\frac{\displaystyle\operatorname*{max}_{i:y^{(i)}=-1}w^{*T}x^{(i)}+\displaystyle\operatorname*{min}_{i:y^{(i)}=1}w^{*T}x^{(i)}}{2}\tag{11}$$

再继续之前，我们先仔细观察一下$(9)$式，也就是最优值$w$关于最优值$\alpha$的函数。假设我们已经通过拟合训练集得到模型的参数，现在想要预测一个新的输入点$x$，那么我们会计算$w^Tx+b$，当且仅当这个值大于$1$时，模型才会给出结论$y=1$。但是通过$(9)$式，这个值也可以表示为：

$$\begin{align}w^Tx+b&=\left(\sum_{i=1}^m\alpha_iy^{(i)}x^{(i)}\right)^Tx+b\tag{12}\\&=\sum_{i=1}^m\alpha_iy^{(i)}\left\langle x^{(i)},x\right\rangle+b\tag{13}\end{align}$$

如果我们已经求出$\alpha_i$，为了做出预测，则只需要按照上式求出$x$与训练集中样本的内积。而且在前面的求解中，我们知道，除了支持向量以为，其余训练样本对应的$\alpha_i$都是零，因此上面的求和中很多项都是零。所以我们只需要求出$x$与支持向量的内积，然后再按照$(13)$式计算并作出预测即可。

为了计算对偶优化问题，我们深入了解了问题的结构，于是仅依靠支持向量的内积就表示出整个算法。在下一节中，我们将继续分析这个特性，进而能够将核方法应用在分类问题。而作为结果得到的**支持向量机（support vector machines）**将会是一个能够在极高维度空间中高效学习的算法。
