
# 第五讲：生成学习算法、高斯判别分析、朴素贝叶斯算法

# 第四部分：生成学习算法（Generative Learning algorithms）

截至目前，我们主要在讨论关于条件$x$下$y$的期望$p(y\mid x;\theta)$的学习模型。比如逻辑回归模型$p(y\mid x;\theta)$中的假设$h_\theta(x)=g(\theta^Tx)$，其中$g$是S型函数。在后面的学习中，我们将介绍一些其他类型的学习算法。

考虑这样一种分类问题，我们想要根据一些动物的特征，区分大象（$y=1$）和小狗（$y=0$）。对于已知的训练集，诸如逻辑回归、感知算法基本上都是尝试找到一条直线，也就是判别边界，以区分大象样本和小狗样本。当需要判别一个新的动物样本时，算法会计算新动物落在判别边界的哪边，进而给出预测结果。

现在来看另一种算法实现。首先，我们观察大象的样本，然后建立一个描述大象的模型；接着，观察小狗样本，建立另一个描述小狗的模型；最后，当需要判断一个新的动物样本是大象还是小狗时，先用样本和大象模型做比对，再用样本和小狗模型做比对，然后看新样本究竟是更像大象还是更像小狗。

那些想要直接学习到$p(y\mid x)$的算法（如逻辑回归），或是想要从输入的特征值空间$\mathcal{X}$直接映射到结果空间${0,1}$上的算法（如感知算法），通常被称作**判别（discriminative）**学习算法。下面我们将要介绍的算法并不对$p(y\mid x)$或$p(y)$建模，这些算法称作**生成（generative）**学习算法，它主要研究给定类别下某种特定特征的概率分布。举个例子，用$y$表示样本是大象（$1$）或是小狗（$0$），则$p(x\mid y=1)$就是类别为大象时，特征的分布模型；同样的，$p(x\mid y=0)$就是类别为小狗时，特征的分布模型。

在对$p(y)$（称为**类先验（class priors）**）和$p(x\mid y)$建模之后，我们就可以使用贝叶斯公式推导出条件$x$下$y$的后验概率：

$$p(y\mid x)=\frac{p(x\mid y)p(y)}{p(x)}$$

上式中的分母来自$p(y)=p(x\mid y=1)p(y=1)+p(x\mid y=0)p(y=0)$（这也是全概率公式的应用），也可以通过前面学到的方式，使用$p(x\mid y)$和$p(y)$表示。实际上，如果是为了做出预测而需要知道$p(y\mid x)$，则并不需要计算出分母的值，因为：

$$\begin{align}\mathrm{arg}\operatorname*{max}_yp(y\mid x)&=\mathrm{arg}\operatorname*{max}_y\frac{p(x\mid y)p(y)}{p(x)}\\&=\mathrm{arg}\operatorname*{max}_yp(x\mid y)p(y)\end{align}$$

$\mathrm{arg}\displaystyle\operatorname*{arg}_y$表示使能够后面跟的式子取最大值的$y$的取值，比如$\mathrm{arg}\displaystyle\operatorname*{min}_x(x-5)^2=5$。这个式子的意思是，求给定$x$的条件下，最有可能的$y$。然后根据贝叶斯公式求最大的$y$，因为$p(x)$独立于$y$，所以分母的取值并不影响式式子取到最大值时$y$的取值。而如果$p(y)$恰好是均匀分布（即对每一种$y$取值，$p(y)$都是一样的，如$p(y=0)=p(y=1)$），那么最后要求的就是使$p(x\mid y)$最大的那个$y$。

## 1. 高斯判别分析（Gaussian discriminant analysis）

我们的第一个生成学习算法的例子是高斯判别分析（GDA）。在这个模型中，我们假设$p(x\mid y)$服从多元正态分布（multivariate normal distribution）。在讨论高斯判别分析之前，我们先来简要回顾一下多元正态分布的特性。

### 1.1 多元正态分布

$n$维多元正态分布也称作多元高斯分布，由一个**期望向量（mean vector）**$\mu\in\mathbb{R}^n$和一个**协方差矩阵（covariance matrix）**$\varSigma\in\mathbb{R}^{n\times n}$参数化，其中$\varSigma\geq 0$是一个对称且半正定的矩阵，通常记为$\mathcal{N}(\mu,\varSigma)$，其概率密度函数为：

$$p(x;\mu,\varSigma)=\frac{1}{(2\pi)^{n/2}\lvert\varSigma\rvert^{1/2}}\exp\left(-\frac{1}{2}(x-\mu)^T\varSigma^{-1}(x-\mu)\right)$$

式中的$\lvert\varSigma\rvert$表示$\varSigma$的行列式。

对于服从$\mathcal{N}(\mu,\varSigma)$的随机变量$X$的期望为：

$$\mathrm{E}[X]=\int_xx\ p(x;\mu,\varSigma)\mathrm{d}x=\mu$$

随即向量$Z$（是一组指[联合分布](http://www.cnblogs.com/vamei/p/3224111.html)的随机变量$(X_1,X_2,\cdots,X_n)$组成的向量）的[协方差（covariance）](https://en.wikipedia.org/wiki/Covariance#Properties)定义为：

$$\mathrm{Cov}(Z,Z)=\mathrm{E}\left[(Z-\mathrm{E}[Z])(Z-\mathrm{E}(Z))^T\right]$$

这个公式也将单一实随机变量的[方差一般化](https://en.wikipedia.org/wiki/Variance#Generalizations)了。方差也可以通过$\mathrm{Cov}(Z,Z)=\mathrm{E}[ZZ^T]-\mathrm{E}[Z](\mathrm{E}[Z])^T$。（两式是等价的，将第一个式子展开即可证明，留意常数的期望不变，而随机变量的期望本身是常数。）如果$X\sim\mathcal{N}(\mu,\varSigma)$，则：

$$\mathrm{Cov}=\varSigma$$

高斯分布概率密度的图像：

<img src="./resource/chapter05_image01.png" width="800" alt="" align=center />

左图为期望为零（$2\times 1$零向量）协方差为$\varSigma=I$（$2\times 2$单位矩阵），这样的高斯分布也叫作标准正态分布（standard normal distribution）。中图的$\varSigma=0.6I$，右图的$\varSigma=2I$。可以看出$\varSigma$越大图像越“分散”，反之则越集中。

再来看几个例子：

<img src="./resource/chapter05_image02.png" width="800" alt="" align=center />

这三幅图的期望仍旧是零，协方差为别为$\varSigma=\begin{bmatrix}1&0\\0&1\end{bmatrix};\quad\varSigma=\begin{bmatrix}1&0.5\\0.5&1\end{bmatrix};\quad\varSigma=\begin{bmatrix}1&0.8\\0.8&1\end{bmatrix}$。当我们将副对角线上的元素值增大时，图像沿$45^\circ$线（因为$x_1=x_2$）方向压缩了。看这三个分布的等高线图时更加明显：

<img src="./resource/chapter05_image03.png" width="800" alt="" align=center />

接下来一组图像：

<img src="./resource/chapter05_image04.png" width="800" alt="" align=center />

这一组图像的协方差分别为：$\varSigma=\begin{bmatrix}1&-0.5\\-0.5&1\end{bmatrix};\quad\varSigma=\begin{bmatrix}1&-0.8\\-0.8&1\end{bmatrix};\quad\varSigma=\begin{bmatrix}3&0.8\\0.8&1\end{bmatrix}$。看前两个图能够发现，当减小协方差矩阵副对角线元素至负数时，图像沿$-45^\circ$线方向压缩了。当我们取更加一般化的协方差矩阵时，等高线变为右图中的椭圆。

来看最后一组例子，这次我们取$\varSigma=I$不变，调整$\mu$：

<img src="./resource/chapter05_image05.png" width="800" alt="" align=center />

上面的图像的期望分别为：$\mu=\begin{bmatrix}1\\0\end{bmatrix};\quad\begin{bmatrix}-0.5\\0\end{bmatrix};\quad\begin{bmatrix}-1\\-1.5\end{bmatrix}$。

### 1.2 高斯判别分析模型

现在来考虑这样一个分类问题，它的输入特征$x$为一系列连续取值的随机变量，这时我们就可以使用高斯判别分析（GDA: Gaussian Discriminant Analysis）模型了，使用多元正态分布对$p(x\mid y)$建模，有：

$$\begin{align}y&\sim\mathrm{Bernoulli}(\phi)\\x\mid y=0&\sim\mathcal{N}(\mu_0,\varSigma)\\x\mid y=1&\sim\mathcal{N}(\mu_1,\varSigma)\end{align}$$

写出分布：

$$\begin{align}p(y)&=\phi^y(1-\phi)^{1-y}\\p(x\mid y=0)&=\frac{1}{(2\pi)^{n/2}\lvert\varSigma\rvert^{1/2}}\exp\left(-\frac{1}{2}(x-\mu_0)^T\varSigma^{-1}(x-\mu_0)\right)\\p(x\mid y=1)&=\frac{1}{(2\pi)^{n/2}\lvert\varSigma\rvert^{1/2}}\exp\left(-\frac{1}{2}(x-\mu_1)^T\varSigma^{-1}(x-\mu_1)\right)\\p(x)&=p(x\mid y=0)p(y=0)+p(x\mid y=1)p(y=1)\end{align}$$

模型中的参数为$\phi,\ \varSigma,\ \mu_0,\ \mu_1$（尽管有两个不同的期望$\mu_0,\ \mu_1$，该模型通常也只是使用一个协方差矩阵$\varSigma$）。参数的对数似然函数为：

$$\begin{align}\mathscr{l}(\phi,\mu_0,\mu_1,\varSigma)&=\log\prod_{i=1}^mp\left(x^{(i)},y^{(i)};\phi,\mu_0,\mu_1,\varSigma\right)\\&=\log\prod_{i=1}^mp\left(x^{(i)}\mid y^{(i)};\mu_0,\mu_1,\varSigma\right)p\left(y^{(i)};\phi\right)\end{align}$$

对比一下前面学过的逻辑回归的对数似然函数：$\mathscr{l}(\theta)=\log\displaystyle\prod_{i=1}^mp\left(y^{(i)}\mid x^{(i)};\theta\right)$，我们称此式为条件似然函数，而上面的式子叫联合似然函数。（化简到第二步时使用了链式法则，后文中有详细介绍。）

为了选择适当的参数使$\mathscr{l}$能够取到最大值，我们需要计算下列参数的最大似然估计：

$$\begin{align}\phi&=\frac{1}{m}\sum_{i=1}^m1\{y^{(i)}=1\}\\\mu_0&=\frac{\sum_{i=1}^m1\{y^{(i)}=0\}x^{(i)}}{\sum_{i=1}^m1\{y^{(i)}=0\}}\\\mu_1&=\frac{\sum_{i=1}^m1\{y^{(i)}=1\}x^{(i)}}{\sum_{i=1}^m1\{y^{(i)}=1\}}\\\varSigma&=\frac{1}{m}\sum_{i=1}^m\left(x^{(i)}-\mu_{y^{(i)}}\right)\left(x^{(i)}-\mu_{y^{(i)}}\right)^T\end{align}$$

将该算法用图像表示如下：

<img src="./resource/chapter05_image06.png" width="400" alt="" align=center />

图中除了训练集样本外，还有已经对两类数据做过拟合的高斯分布的等高线图。注意到这两个高斯分布的等高线图形状、方向都是是一样的，因为我们使用了同一个协方差矩阵$\varSigma$，只是期望$\mu_0,\ \mu_1$不同。图中的直线就是由$p(y=1\mid x)=0.5$得到的判定边界了。在边界的一侧，样本将得到$y=1$的预测，另一侧则会得到$y=0$的预测。

### 1.3 讨论：高斯判别分析与逻辑回归

高斯判别分析模型与逻辑回归有着有趣的联系。如果我们将$p(y=1\mid x;\phi,\mu_0,\mu_1,\varSigma)$看作是关于$x$的函数，则有：

$$p(y=1\mid x;\phi,\mu_0,\mu_1,\varSigma)=\frac{1}{\exp\left(\theta^Tx\right)}$$

式中的$\theta$是一个关于$\phi,\mu_0,\mu_1,\varSigma$的函数。（同前面一样，我们令$x_0^{(i)}=1$，于是的$x^{(i)}$变为$n+1$维向量。）这正是逻辑回归中的假设函数，用于对$p(y=1\mid x)$建模。（也就是说，如果我们在使用高斯判别建模时假设$x\mid y\sim\mathrm{Gaussian}$，则可以推导出$p(y\mid x)$是一个逻辑函数。）

通常来讲，针对同一训练集，高斯判别分析和逻辑回归将给出不同的判定边界，那么，我们该如何在这两种算法里取舍？

在上面我们证明了，如果$p(x\mid y)$服从多元高斯分布（各元变量使用同一$\varSigma$）则$p(y\mid x)$是一个逻辑函数。但是反过来则不一定，如果$p(y\mid x)$是一个逻辑函数，并不能推导出$p(x\mid y)$是多元高斯分布。这说明，相对于逻辑回归，高斯判别分析对数据做出了*更强*的模型假设（我们假设了在$y$条件下$x$服从高斯分布，这是一个很强的假设）。也就是说，在假设模型正确（或$y\mid x$大致服从高斯分布）的前提下，同逻辑回归相比，高斯判别分析能够对数据做出更好的拟合，得到更好的模型（我们显示的认定了训练集数据服从高斯分布，则如果我们在算法中使用了这个假设，那么这个算法的表现将更好，因为这个算法利用了跟多关于数据的信息，算法预先就“知道”数据服从高斯分布）。特别是当$p(x\mid y)$确实大致服从多元高斯分布（各元变量使用同一$\varSigma$）时，高斯判别分析是**渐近有效（asymptotically efficient）**的。非正式的讲，对于较大的的有限训练集（$m$足够大），没有任何算法能够比高斯判别分析做的更好。（即度量算法对$p(y\mid x)$评估的准确度）。特别的，对于本例中的数据，高斯判别分析确实是更好的算法。对于更加普通的例子，即使面对较小的训练集，高斯判别分析也通常更优秀（因为我们预先做了足够强的假设，即使仅使用非常少的数据，高斯判别分析也能得出一个还不错的模型。）

但从另一个角度讲，对于很弱的假设，逻辑回归更加健壮，它对异常模型假设更加不敏感。有许多不同的假设最终都能以逻辑函数的形式推导出$p(y\mid x)$。举个例子，如果$x\mid y=0\sim\mathrm{Poisson}(\lambda_0);\quad x\mid y=1\sim\mathrm{Poisson}(\lambda_1)$，则$p(y\mid x)$也将表达为逻辑函数的形式。可见逻辑回归对服从泊松分布的数据依然有着良好的效果，但是如果使用高斯判别分析，使用高斯分布拟合非高斯分布的数据，那么得出的结果将变得不可靠，也就是使用高斯判别分析的效果会大打折扣。（如果在不确定$x\mid y$的分布情况的情况下，那么逻辑回归的表现可能更好。因为逻辑回归做的预先假设更少，所以对模型假设方面更为健壮，不过与高斯模型比起来，可能会需要更多的训练样本。）

（实际上，更一般的有：$x\mid y=0\sim\mathrm{ExponentialFamily}(\eta_0);\quad x\mid y=1\sim\mathrm{ExponentialFamily}(\eta_1)$，则能够推导出$p(y\mid x)$会是一个逻辑函数。也就是说，即使$x\mid y$服从伽马、贝塔等任何指数族中的分布时，所得的后验概率$p(y=1\mid x)$都将是逻辑函数。）

综上，当模型假设正确或近似正确时，高斯判别分析能够做出更强的模型假设，且对训练数据效率更高（即使使用少量样本依然能够得到较好的效果，收敛更加迅速）。逻辑回归得到的假设较弱，对偏离模型假设的数据表现出良好的健壮性（更加抗噪音）。特别是对于明显不属于高斯分布，在较大的有限训练集支持下，逻辑回归得到的结果通常都比高斯判别分析更好。也正是因为这个原因，实际使用中我们更加偏向于选择逻辑回归。（一些关于判别模型与生成模型的对比同样适用于我们接下来介绍的朴素贝叶斯算法，不过朴素贝叶斯算法仍然不失为一种良好的、应用广泛的分类算法。）

## 2. 朴素贝叶斯算法（Naive Bayes）

在高斯判别分析中，输入特征$x$是一个连续的实向量。现在我们来看另一种学习算法，其输入特征$x$是离散的。

继续考虑前面关于垃圾邮件过滤器的例子，这次我们希望判断一封邮件是强推广告（垃圾）还是正常邮件。如果实现了这个功能，我们的邮件阅读器就可以实现自动过滤或者将垃圾邮件放在不同的文件夹中了。邮件分类问题属于一个叫做**文本分类（text classification）**问题的大类。

假设有训练集（标有垃圾或正常标签的邮件集合），我们通过选取特征$x_i$描述邮件，进而设计我们自己的邮件过滤器。

邮件特征值的个数为我们“邮件词典”中单词的数量，将这些特征值组成向量。如果某邮件中含有词典里第$i$个词，则该邮件特征值向量的第$i$个分量置为$1$，即$x_i=1$，反之$x_i=0$，比如特征值向量$x=\begin{array}{|c|c}1&\mathrm{a}\\0&\mathrm{aardvark}\\0&\mathrm{aardwolf}\\\vdots&\vdots\\1&\mathrm{buy}\\\vdots&\vdots\\0&\mathrm{zygmurgy}\end{array}$，表示的该邮件中包含“a”和“buy”，而不包含“aardvark”、“aardwolf”和“zygmurgy”。

编码在特征值向量中的词集称为**词汇表（vocabulary）**，所以邮件特征值向量的维数和词汇表的大小是一致的。（为了节省内存、CPU资源，我们会精心设计词汇表，对诸如“a”、“the”、“of”、“and”等使用频率非常高，而对邮件内容影响很小的词（这些词几乎会出现在所有文章中，也称为**stop words**），我们通常将他们排除在词典之外。）

选定邮件特征值向量之后，就可以着手设计生成学习算法了，我们需要对$p(x\mid y)$建模。但是，假设我们的词汇表有$50000$个条目，则$x\in\{0,1\}^{50000}$（即$x$是一个由由$0,\ 1$组成的$50000$维向量），如果我们按正常步骤使用上一讲的softmax回归对$x$建模，则会得到一个有$2^{50000}$种可能输出的模型，为了给每一种可能的输出设定一个参数，接下来还需要设计一个$(2^{50000}-1)$维的参数向量……很明显，参数过多了。

为了给$y(x\mid y)$建模，我们将做出一个很强（严格）的假设：设$x^{(i)}$对于给定的条件$y$是独立的，这个假设叫做**朴素贝叶斯假设（Naive Bayes (NB) assumption）**，而根据此假设得到的算法称为**朴素贝叶斯分类器（Naive Bayes classifier）**。举个例子，设$y=1$表示垃圾邮件，“buy”是词汇表中第$2087$个词，“price”是词汇表中第$39831$个词，于是我们做出假设：一旦给定$y=1$（则该邮件已经被确认是垃圾邮件），则$x_{2087}$（即“buy”是否出现在邮件中）不会对我们关于$x_{39831}$（“price”是否出现）的见解构成任何影响。更加形式化的表达：$p(x_{2087}\mid y)=p(x_{2087}\mid y,x_{39831})$（“price”是否出现并不影响条件$y$下“buy”出现的概率）。（注意，该式并*不是*用来描述$x_{2087},\ x_{39831}$相互独立这一假设，描述这一假设的写法应为$p(x_{2087})=p(x_{2087}|x_{39831})$；我们这里做出的假设是在*给定的条件$y$下*$x_{2087}$与$x_{39831}$相互独立。）

于是有：

$$\begin{align}p(x_1,\cdots,x_{50000}\mid y)&=p(x_1\mid y)p(x_2\mid y,x_1)p(x_3\mid y,x_1,x_2)\cdots\ p(x_{50000}\mid y,x_1,\cdots,x_{49999})\\&=p(x_1\mid y)p(x_2\mid y)p(x_3\mid y)\cdots p(x_{50000}\mid y)\\&=\prod_{i=1}^n p(x_i\mid y)\end{align}$$

第一个等式使用了[联合分布的链式法则](https://en.wikipedia.org/wiki/Joint_probability_distribution#Density_function_or_mass_function)（参见[链式法则](https://en.wikipedia.org/wiki/Chain_rule_%28probability%29)），第二个等式使用了前面的的朴素贝叶斯假设。尽管朴素贝叶斯假设是一个很强的假设条件，但是该算法得到的模型对于很多问题都有良好的效果。

我们的模型由参数$\phi_{i\mid y=1}=p(x_i=1\mid y=1),\ \phi_{i\mid y=0}=p(x_i=1\mid y=0),\ \phi_y=p(y=1)$确定。与以往的步骤一样，有了训练集$\left\{\left(x^{(i)},y^{(i)}\right);i=1,\cdots,m\right\}$，我们可以写出关于参数的联合似然函数：

$$\begin{align}\mathcal{L}\left(\phi_y,\phi_{j\mid y=0},\phi_{j\mid y=1}\right)&=\prod_{i=1}^mp\left(x^{(i)},y^{(i)}\right)\end{align}$$

求$\mathcal{L}$关于$\phi_y,\phi_{j\mid y=0},\phi_{j\mid y=1}$的最大值，得到三个参数的最大似然估计：

$$\begin{align}\phi_{j\mid y=1}&=\frac{\sum_{i=1}^m1\left\{x_j^{(i)}=1\land y^{(i)}=1\right\}}{\sum_{i=1}^m1\left\{y^{(i)}=1\right\}}\\\phi_{j\mid y=0}&=\frac{\sum_{i=1}^m1\left\{x_j^{(i)}=1\land y^{(i)}=0\right\}}{\sum_{i=1}^m1\left\{y^{(i)}=0\right\}}\\\phi_y&=\frac{\sum_{i=1}^m1\left\{y^{(i)}=1\right\}}{m}\end{align}$$

上式中的$\land$表示逻辑“且”。这些参数都有很自然的表达，比如$\phi_{j\mid y=1}$就是指含有单词$x_j$的垃圾邮件在所有垃圾邮件中出现的概率。

求参数拟合后，若要对具有特征$x$的新样本进行预测我们就可以使用贝叶斯公式进行预测了：

$$
\begin{align}p(y=1\mid x)&=\frac{p(x\mid y=1)p(y=1)}{p(x)}\\
&=\frac{\left(\prod_{i=1}^np(x_i\mid y=1)\right)p(y=1)}{\left(\prod_{i=1}^np(x_i\mid y=1)\right)p(y=1)+\left(\prod_{i=1}^np(x_i\mid y=0)\right)p(y=0)}\end{align}
$$

然后选取后验概率较大的那一类即可做出预测。

到目前为止，我们学习的朴素贝叶斯算法，主要针对二元特征值$x_i$（取$\{0,1\}$），下面介绍的算法适用于一般化的在$1,2,\cdots,k$取值的特征值$x_i$。

我们只需要将$p(x_i\mid y)$从伯努利分布改为多项分布即可，即使存在一些取连续值的特征值（比如在前几讲例子中提到的的“房屋实用面积”），我们也可以把它**离散化（discretize）**（方法很简单，就是将连续的取值分区间映射在一个较小的集合中），进而继续使用朴素贝叶斯算法。举个例子：我们使用特征值$x_i$描述“实用面积”，将连续值离散化的方法为：


$$\begin{array}{c|c|c|c|c|c}\mathrm{Living area(sq.\ feet)}&\lt 400&400-800&800-1200&1200-1600&\gt 1600\\\hline x_i&1&2&3&4&5\end{array}$$

因此，如果有面积为$890$平尺实用面积的房子，我们就将它的特征值$x_i$设为$3$。然后像前面一样，继续使用朴素贝叶斯算法，使用多项分布对$p(x_i\mid y)$建模。如果原始特征值取的是连续值，不能通过使用多项分布来建立良好的模型时，则可以离散化这些特征值，然后选择朴素贝叶斯算法（而不是高斯判别分析）通常会得到较好的分类器模型。

## 2.1 拉普拉斯平滑（Laplace smoothing）

上面介绍的朴素贝叶斯算法，对于很多问题都有良好的效果，不过我们只要做一个小改动，就能让它变得更好，特别是对文本分类问题。现在我们来简单的演示一个在当前的朴素贝叶斯算法下可能会遇到的“漏洞”，之后我们会讨论如何修补这个“漏洞”。

仍旧是垃圾邮件分类器，假设我们在本课程结束（CS229）之后的结题项目中表现优秀，于是打算在2003年六月将我们的项目投稿在NIPS会议上。（NIPS是最顶尖的机器学习会议之一，其投稿截止时间通常在六月底七月初。）因为这个原因，我们会在截止日期前的邮件中使用“nips”这个词语，同时也会在这段时间开始收到带有“nips”这个词语的邮件。不过，这是我们在NIPS上的第一次投稿，此前我们从没在邮件中使用过“nips”。我们说的更绝对一些，“nips”从未出现在我们的“垃圾/正常邮件训练集”中。假设“nips”是词典中的第35000个单词，于是我们的贝叶斯邮件过滤器将计算参数$\phi_{35000\mid y}$的最大似然估计：

$$\begin{align}\phi_{35000\mid y=1}&=\frac{\sum_{i=1}^m1\left\{x_{35000}^{(i)}=1\land y^{(i)}=1\right\}}{\sum_{i=1}^m1\left\{y^{(i)}=1\right\}}=0\\\phi_{35000\mid y=0}&=\frac{\sum_{i=1}^m1\left\{x_{35000}^{(i)}=1\land y^{(i)}=0\right\}}{\sum_{i=1}^m1\left\{y^{(i)}=0\right\}}=0\end{align}$$

也就是说，如果以前（训练集）从未见过“nips”这个词，则模型认为该词在任何一类邮件中出现的概率皆为零。于是，当我们需要预测一封带有“nips”的邮件是否为垃圾邮件时，模型会计算其后验概率：

$$\begin{align}p(y=1\mid x)&=\frac{p(x\mid y=1)p(y=1)}{p(y=1)}\\&=\frac{\left(\prod_{i=1}^np(x_i\mid y=1)\right)p(y=1)}{\left(\prod_{i=1}^np(x_i\mid y=1)\right)p(y=1)+\left(\prod_{i=1}^np(x_i\mid y=0)\right)p(y=0)}\\&=\frac{0}{0}\end{align}$$

容易看出，$\prod_{i=1}^np(x_i\mid y)$项中有$p(x_{35000}\mid y)=0$作为乘数，所以最终得到$\frac{0}{0}$，然后就不知所措了。

扩展这个问题，从统计的角度讲，“认为从未发生过（即训练集中没有的）的事件不可能发生”是一个坏点子。带着这个问题，我们来看一下在$\{1,\cdots,k\}$中取值的多项随机变量$z$的期望估计：我们可以使用$\phi_i=p(z=i)$参数化这个多项分布，对于有$m$个独立观测集合$\left\{z^{(1)},\cdots,z^{(i)}\right\}$，参数的极大似然估计为：

$$\phi_j=\frac{\sum_{i=1}^m1\left\{z^{(i)}=j\right\}}{m}$$

同我们前面遇到的问题一样，如果对参数使用极大似然估计，则某些$\phi_j$可能会为零。为了避免这个现象，我们使用**拉普拉斯平滑（Laplace smoothing）**，将上面的估计替换为：

$$\phi_j=\frac{\sum_{i=1}^m1\left\{z^{(i)}=j\right\}+1}{m+k}$$

可以看到，我们给分子加了$1$，给分母加了$k$，这样，有$\sum_{j=1}^k\phi_j=1$仍然成立（这只是我们需要的性质，因为$\phi_j$是各可能结果的概率估计，其和必须为$1$）；同时，对于所有$\phi_j$有$\phi_j\neq0$成立。在特定条件下（存在争议），拉普拉斯平滑给力参数$\phi_j$一个理想的估计。

回到我们的朴素贝叶斯分类器，对参数加入拉普拉斯平滑，得到如下参数估计：

$$\begin{align}\phi_{j\mid y=1}&=\frac{\left(\sum_{i=1}^m1\left\{x_j^{(i)}=1\land y^{(i)}=1\right\}\right)+1}{\left(\sum_{i=1}^m1\left\{y^{(i)}=1\right\}\right)+2}\\\phi_{j\mid y=0}&=\frac{\left(\sum_{i=1}^m1\left\{x_j^{(i)}=1\land y^{(i)}=0\right\}\right)+1}{\left(\sum_{i=1}^m1\left\{y^{(i)}=0\right\}\right)+2}\end{align}$$

（在实际问题中，我是否对$\phi_y$做拉普拉斯平滑并不重要，因为在结果集中，对于于正常或垃圾邮件的概率$p(y=1),p(y=0)$，应该都不会出现零值。）

如此，我们的垃圾邮件分类器在第一次遇到”nips“时，也可以做出有意义的预测了。