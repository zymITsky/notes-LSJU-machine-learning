
# 第六讲：事件模型、函数间隔与几何间隔

接着上一讲，进一步讨论朴素贝叶斯算法及事件模型。

## 2.2 文本分类的事件模型

在结束生成学习算法前，我们再来介绍一种针对文本分类的模型。虽然前面介绍的朴素贝叶斯算法对于很多分类问题都要良好的效果，不过对于文本分类，我们有一种更好的模型。

在特定语境的文本分类中，朴素贝叶斯算法使用了**多元伯努利事件模型（multi-variate Bernoulli event model）**（特征值按照伯努利试验取$x_i\in\{0,1\}$，特征值向量长度为字典长度）。回忆一下上一讲的知识，在这个模型中，我们首先假设下一封邮件的发送方已经确定，一个随机的邮件发送人（可能是垃圾邮件制造者，也可能只是普通联系人，是由我们确定的条件概率中的条件）；接着我们假设，发送者会遍历词典，独立的决定是否将单词$x_i$写入邮件（该词语相对于其他词语独立），而且该词语服从概率$p(x_i=1\mid y)=\phi_{i\mid y}$（如果是垃圾邮件制造者则可能更倾向于发送带有“buy”、“sales”、“discount”等单词的 邮件，而普通联系人则会选择更正常的词语分布发送正常邮件）。于是我们得到，$p(x\mid y)=\prod_{i=1}^np(x_i\mid y)$和$p(y)$，再根据$\mathrm{arg}\displaystyle\operatorname*{max}_yp(x\mid y)=\mathrm{arg}\displaystyle\operatorname*{max}_yp(x\mid y)p(y)$最终得到该邮件是垃圾邮件的概率：$p(y)\prod_{i=1}^np(x_i\mid y)$。这个模型存在一些小问题，比如，它不能讲词语的出现的次数反应在输入特征值向量中，如果一封邮件多次出现“buy”这个词，那么此邮件确实更有可能是垃圾邮件。

下面介绍的模型是朴素贝叶斯算法算法的简单变形，称为**多项事件模型（multinomial event model）**。接下来，我们将使用不同的记法和不同的特征值描述邮件。使用$x_i$表示邮件中的第$i$个词语，$x_i$在是一个在$\{1,\cdots,\lvert V\rvert\}$取值的整数，而$\lvert V\rvert$就是词汇表（词典）的大小。于是，一封邮件被我们表示为$(x_1,x_2,\cdots,x_n)$向量，注意，对于不同的邮件$n$是不同的。举个例子，对于一封以“A NIPS”开头的邮件，有$x_1=1$（“a”是词典的第一个单词）、$x_2=35000$（假设“nips”是词典的第35000个单词）。

在多项事件模型中，我们像以前一样假设邮件发送方已经由一个随机过程确定（根据$p(y)$），可能是垃圾邮件制造者，也可能是普通联系人；接着，发件人从某多项分布（$p(x_i\mid y)$）中选出一个词$x_1$；然后根据相同的多项分布选出第二个词$x_2$（独立于$x_1$），继续重复这个动作选出$x_3,x_4,\cdot,x_n$；最后，邮件编写结束。因此，该邮件是垃圾邮件的概率为$p(y)\prod_{i=1}^np(x_i\mid y)$，这个式子和前面的完全一样，但是请注意，这里的特征值$x_i$是邮件第$i$个单词，其取值不再是多元伯努利事件模型中的$\{0,1\}$了，它现在是一个多项分布。

多项事件模型的参数为：$\phi_y=p(y)$（同以前一样），$\phi_{k\mid y=1}=p(x_j=k\mid y=1),\ \phi_{k\mid y=0}=p(x_j=k\mid y=0)$（对任意$j$）。注意到这里我们假设了对于任意$j$都有其$p(x_j\mid y)$相等，即在$y$条件下关于某词语的概率分布与该词语出现在邮件里的位置（$j$）无关。

对于训练集$\left\{\left(x^{(i)},y^{(i)}\right);i=1,\cdots,m\right\}$（其中$x^{(i)}=\left(x_1^{(i)},x_2^{(i)},\cdots,x_{n_i}^{(i)}\right)$代表训练集第$i$封样本邮件特征值向量，邮件共有$n_i$个单词），参数的似然函数为：

$$\begin{align}\mathcal{L}\left(\phi_y,\phi_{k\mid y=0},\phi_{k\mid y=1}\right)&=\prod_{i=1}^mp\left(x^{(i)},y^{(i)}\right)\\&=\prod_{i=1}^m\left(\left(\prod_{j=1}^{n_i}p\left(x_j^{(i)}\mid y;\phi_{k\mid y=1},\phi_{k\mid y=0}\right)\right)p\left(y^{(i)};\phi_y\right)\right)\end{align}$$

最大化似然函数将得到各参数的最大似然估计：

$$\begin{align}\phi_{k\mid y=1}&=p(x_j=k\mid y=1)&=\frac{\sum_{i=1}^m\sum_{j=1}^{n_i}1\left\{x_j^{(i)}=k\land y^{(i)}=1\right\}}{\sum_{i=1}^m1\left\{y^{(i)}=1\right\}n_i}\\\phi_{k\mid y=0}&=p(x_j=k\mid y=0)&=\frac{\sum_{i=1}^m\sum_{j=1}^{n_i}1\left\{x_j^{(i)}=k\land y^{(i)}=0\right\}}{\sum_{i=1}^m1\left\{y^{(i)}=0\right\}n_i}\\\phi_y&=p(y=1)&=\frac{\sum_{i=1}^m1\left\{y^{(i)}=1\right\}}{m}\end{align}$$

我们对$\phi_{k\mid y=0},\phi_{k\mid y=1}$应用拉普拉斯平滑（在实际问题中应用拉普拉斯平滑通常可以得到更好的模型），即在分子上加一，分母上加$\lvert V\rvert$，得到：

$$\begin{align}\phi_{k\mid y=1}&=\frac{\left(\sum_{i=1}^m\sum_{j=1}^{n_i}1\left\{x_j^{(i)}=k\land y^{(i)}=1\right\}\right)+1}{\left(\sum_{i=1}^m1\left\{y^{(i)}=1\right\}n_i\right)+\lvert V\rvert}\\\phi_{k\mid y=0}&=\frac{\left(\sum_{i=1}^m\sum_{j=1}^{n_i}1\left\{x_j^{(i)}=k\land y^{(i)}=0\right\}\right)+1}{\left(\sum_{i=1}^m1\left\{y^{(i)}=0\right\}n_i\right)+\lvert V\rvert}\end{align}$$

在处理文本分类问题时，多项事件模型通常比原始的朴素贝叶斯算法效果更好，一个可能的原因是因为它考虑了每个单词出现的次数。

尽管朴素贝叶斯分类器不是最好的分类算法，但它的效果一般都非常好，再加上它简单且易于实现的特性，我们通常用它作为“首选试验算法”。（使用朴素贝叶斯算法最终会得到一个逻辑函数形式的后验分布，也就是说，朴素贝叶斯算法也属于指数分布族，它仍然是一个线性分类器。）

# 第五部分：支持向量机

接下来的几个掌机会介绍支持向量机学习算法（SVM），它通常被认为是最好的现成监督学习算法之一（很多人认为它是最好的）。为了详细介绍支持向量机，我们需要先了解间隔（margin）的概念，以及使用大间隙（gap）分割数据的想法。接着我们将介绍优化间隔分类器，进而讨论一下语言歧义（language duality）现象。然后介绍支持向量机的核心——如何在一个极高维特征值向量空间（比如无限维）上高效的应用支持向量机算法。最后我们将介绍序列最小优化算法（SMO: sequential minimal optimization）算法——一个更优化的支持向量机的实现。

## 1. 间隔：直观概念

我们从间隔开始讨论支持向量机算法，本节将给出“间隔”以及算法对预测的“信心”的直观概念，在第三节我们会进一步形式化这些概念。

回顾前面的逻辑回归，假设函数的模型$h_\theta(x)=g\left(\theta^Tx\right)$将给出关于$p(y=1\mid x;\theta)$的概率预测。当且仅当$h_\theta(x)\geq 0.5$即输入满足$\theta^Tx\geq 0$时，模型给出关于输入的预测为$1$。对于一个正训练样本（即$y=1$），$\theta^Tx$越大，则$h_\theta(x)=p(y=1\mid x;w,b)$，也就意味着我们将其预测为$1$的“信心”越高。因此，严谨的说，如果$\theta^Tx\gg 0$，则算法预测其为$1$的信心就会非常大。同样的，对于逻辑回归，如果$\theta^Tx\ll 0$，则算法预测其为$0$的信心就会非常大。对于给定的训练集，我们可以说，如果拟合参数$\theta$能够使得所有$y^{(i)}=1$的样本满足$\theta^Tx^{(i)}\gg0$，使所有$y^{(i)}=0$的样本满足$y^{(i)}=0$，则称这是一个良好的拟合，因为这反映出该拟合对分类结果的“信心”很足。所以，“信心”是一个很好的指标，后面我们将使用函数间隔形式化这个指标。

另一种直观的表达，如下图，其中x代表正训练样本，o代表负训练样本，直线就是判别边界（由$\theta^Tx=0$给出，也叫**分类超平面（separating hyperplane）**）。

<img src="./resource/chapter06_image01.png" width="400" alt="" align=center />

图中的三个点A、B、C，点A距离判别边界非常远，如果求关于点A的预测，我们会非常确定其值为$y=1$。相反，点C距离判别边界很近，虽然它在判别边界$y=1$一侧，但是如果判别边界稍有变动，它就有可能被放在$y=0$一侧。因此，我们对点A预测的信心强于点C。而点B则在两者之间，通常来说，如果点距离判别边界越远，模型做出预测的信心就越强。也就是说，如果对于给定训练集，可以找到一条能够准确并可信（即样本距离边界很远）的预测所有训练样本的判别边界，则称这个拟合是良好的。我们在后面将使用几何间隔形式化这个概念。

## 2. 标记法

为了更加简洁的介绍支持向量机，我们需要先引入一种新的标记。考虑使用线性分类器解决“特征为$x$目标为$y$”的二元分类问题。这次，我们使用$y\in\{-1,1\}$来标记两个分类（而不是之前的$y\in\{0,1\}$），再使用参数向量$w,b$代替之前的参数向量$\theta$，于是我们现在将分类器写为：

$$h_{w,b}(x)=g\left(w^Tx+b\right)$$

此处，$g(z)=\begin{cases}1 &z\gt 0\\-1 &z\lt 0\end{cases}$，而$w,b$的记法可以将截距项与特征值的系数分开来记（也不再向特征值向量$x$中添加$x_0=1$分量），即$b$用来代替原来的$\theta_0$，而$w$用来代替原来的$\begin{bmatrix}\theta_1&\cdots&\theta_n\end{bmatrix}^T$。

还需要注意的是，根据函数$g$的定义，分类器将直接给出$-1$或$1$的结果（类似感知算法），省略了估计$y=1$的概率的步骤。

## 3. 函数间隔及几何间隔

本节将形式化函数间隔及几何间隔。对于给定的训练样本$\left(x^{(i)},y^{(i)}\right)$，我们定义关于此训练样本函数间隔的超平面$(w,b)$为：

$$\hat{\gamma}^{(i)}=y^{(i)}\left(w^Tx^{(i)}+b\right)$$

上式可以解释为，当$y^{(i)}=1$时，为了得到一个较大的函数间隔（即为了使预测有较高的的可信度及准确性），我们需要$w^Tx+b$取一个较大的正数（$w^Tx+b\gg 0$）；反之，当$y^{(i)}=-1$时，我摸需要$w^Tx+b$取一个较大的负数（$w^Tx+b\ll 0$）。此外，如果$y^{(i)}\left(w^Tx+b\right)\gt 0$，则说明关于此样本的预测是正确的。因此，较大的函数间隔意味着较高的可信度和准确性。

对于一个在$g\in\{-1,1\}$取值的线性分类器，有一个性质导致其函数间隔不能有效的反映预测的可信度：对于给定的$g$，如果将$(w,b)$替换为$(2w,2b)$，即$g\left(w^Tx+b\right)$变为$g\left(2w^Tx+2b\right)$，我们会发现分类超平面$h_{w,b}(x)$并不会改变，也就是说$h_{w,b}(x)$只关心$w^Tx+b$的正负，而不关心其大小。但是将$(w,b)$变为$(2w,2b)$相当于给函数间隔乘了系数$2$，于是我们发现，如果通过改变$w,b$的取值，我们可以让函数间隔变得很大，然而分类超平面并没有改变，所以单纯的通过这种方式改变函数间隔的大小没有什么实质意义。直觉告诉我们，应该引入一种标准化条件，比如令$\lVert w\rVert_2=1$，即把$(w,b)$变为$\left(\frac{w}{\lVert w\rVert_2},\frac{b}{\lVert w\rVert_2}\right)$，我们在后面会继续讨论这种方法。

对于给定的训练集$S=\left\{\left(x^{(i)},y^{(i)}\right);i=1,\cdots,m\right\}$，关于$S$以$(w,b)$为参数的函数间隔$\hat\gamma$定义为取所以独立的训练样本中最小的那个函数间隔（即取最坏的一个样本的情况）：

$$\hat\gamma=\operatorname*{min}_{i=1,\cdots,m}\hat\gamma^{(i)}$$

接下来我们讨论几何间隔，考虑下图：

<img src="./resource/chapter06_image02.png" width="400" alt="" align=center />

直线为$(w,b)$确定的判定边界（即超平面$w^Tx+b=0$），向量$w$正交于分类超平面（在超平面上任取两点$P_1=(x_1,\cdots,x_n), P_2==(x'_1,\cdots,x'_n)$，则两点满足平面方程组$\begin{cases}w^TP_1+b=0\\w^TP_2+b=0\end{cases}$，两式相减得$w^T(P_1-P_2)=0$，即$w^T$正交于该超平面上任意向量，所以$w^T$为超平面法向量）。观察点$A$，设点$A$为训练样本$(x^{(i)},y^{(i)}=1)$，它到判定边界的距离$\gamma^{(i)}$即为线段$AB$的长度。

如何计算$\gamma^{(i)}$？注意到$\frac{w}{\lVert w\rVert}$是标准化后的$w$向量，点$A$为$x^{(i)}$，则点$B$为$x^{(i)}-\gamma^{(i)}\cdot\frac{w}{\lVert w\rVert}$。点$B$在判定边界上，而判定边界上的所有点都满足方程$w^Tx+b=0$，则：

$$w^T\left(x^{(i)}-\gamma^{(i)}\frac{w}{\lVert w\rVert}\right)+b=0$$

解出$\gamma^{(i)}$得：（注：$w^Tw=\lVert w\rVert^2$）

$$\gamma^{(i)}=\frac{w^Tx^{(i)}+b}{\lVert w\rVert}=\left(\frac{w}{\lVert w\rVert}\right)^Tx^{(i)}+\frac{b}{\lVert w\rVert}$$

这就是正训练样本$A$被正确的分在判别边界$y=1$一侧的情形，更一般的，我们定义关于样本$\left(x^{(i)},y^{(i)}\right)$以$(w,b)$为参数的函数间隔为：

$$\gamma^{(i)}=y^{(i)}\left(\left(\frac{w}{\lVert w\rVert}\right)^Tx^{(i)}+\frac{b}{\lVert w\rVert}\right)$$

可以看出，如果$\lVert w\rVert=1$，则函数间隔等于几何间隔（$\hat\gamma^{(i)}=\gamma^{(i)}$），这就是两种间隔的关系（$\hat\gamma^{(i)}=\frac{\gamma^{(i)}}{\Vert w\Vert}$）。与函数间隔一样，如果改变参数$(w,b)$为$(2w,2b)$，则几何间隔不会改变。这个性质在后面会很方便，利用改变参数$(w,b)$的大小不影响间隔，我们可以在拟合参数时引入$w$的限制条件，比如令$\lVert w\rVert=1$，或$\lvert w_1\rvert=5$，或$\lvert w+b\rvert+\lvert b\rvert=2$，诸如这种限制条件都可以通过给参数$(w,b)$乘以一个适当的系数得到。

对于给定的训练集$S=\left\{\left(x^{(i)},y^{(i)}\right);i=1,\cdots,m\right\}$，也有关于$S$以$(w,b)$为参数的几何间隔$\gamma$定义为取所以独立的训练样本中最小的那个几何间隔（即取最坏的一个样本的情况）：

$$\gamma=\operatorname*{min}_{i=1,\cdots,m}\gamma^{(i)}$$

另外，几何间隔实际上就是点到超平面的距离，高中时学过的点$\left(x^{(i)},y^{(i)}\right)$到直线$ax+by+c=0$的距离为：

$$d\left(x^{(i)},y^{(i)}\right)=\frac{\lvert ax^{(i)}+by^{(i)}+c\rvert}{\sqrt{a^2+b^2}}$$

推广到高维就是上面的几何间隔，而函数间隔就是未标准化的几何间隔。

最后，最大间隔分类器（maximum margin classifier，可以被看做是支持向量机的前身），实际上就选择特定的$w,b$使几何间隔最大化：

$$\begin{align}\displaystyle\operatorname*{max}_{w,b}&\quad\gamma\\\mathrm{s.t.}&\quad y^{(i)}\left(\left(\frac{w}{\lVert w\rVert}\right)^Tx^{(i)}+\frac{b}{\lVert w\rVert}\right)\end{align}$$

注：$\mathrm{s.t.}$是“subject to”的缩写，意为“受限于”。