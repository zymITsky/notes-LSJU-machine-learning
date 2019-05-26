
# 第八讲：核方法、序列最小优化算法

先回顾上一讲的内容：

我们写出凸优化问题（假设数据都是线性可分的）：

$$\begin{align}\displaystyle\operatorname*{min}_{w,b}&\quad\frac{1}{2}\lVert w\rVert^2\\\mathrm{s.t.}&\quad y^{(i)}\left(w^Tx^{(i)}+b\right)\geq 1,\quad i=1,\cdots,m\end{align}$$

对于给定的训练集，算法会找到该数据集的最优间隔分类器，可以使训练样本的几何间隔最大化，进而推导出了这个问题的对偶问题：

$$\begin{align}\displaystyle\operatorname*{max}_{\alpha}&\quad W(\alpha)=\sum_{i=1}^m\alpha_i-\frac{1}{2}\sum_{i,j=1}^m\alpha_i\alpha_jy^{(i)}y^{(j)}\left\langle x^{(i)},x^{(j)}\right\rangle\\\mathrm{s.t.}&\quad \alpha\geq 0,\quad i=1,\cdots,m\\&\quad\sum_{i=1}^m\alpha_iy^{(i)}=0\end{align}$$

在推导过程中，我们也得到$w$：

$$w=\sum_{i=1}^m\alpha_iy^{(i)}x^{(i)}$$

于是，当我们需要对样本进行预测时，我们需要计算输入向量的假设值：

$$\begin{align}h_{w,b}(x)&=g\left(w^Tx+b\right)\\&=g\left(\left(\sum_{i=1}^m\alpha_iy^{(i)}x^{(i)}\right)^Tx+b\right)\\&=g\left(\sum_{i=1}^m\alpha_iy^{(i)}\left\langle x^{(i)},x\right\rangle+b\right)\end{align}$$

接下来，我们介绍核方法。

## 7. 核方法（Kernels）

回忆关于线性回归的讨论，我们用输入$x$表示“出租房屋的实用面积”，然后考虑使用诸如$x,x^2,x^3$的特征值来拟合一个三次函数。为了区分这两组变量，我们称“最初的”输入值（即本例中的“实用面积”$x$）为问题的**输入属性（input attributes）**；而当“输入属性”映射在一组新的量中，进而转给学习算法时，我们称这组新的量为**输入特征（input features）**（不过，不同的作者会使用不同的名称来指代这两个概念，我们将在后面的学习中继续保持这种称谓）。我们使用$\phi$表示**特征映射（feature mapping）**，即属性到特征的映射，比如在本例中有：

$$\phi(x)=\begin{bmatrix}x\\x^2\\x^3\end{bmatrix}$$

相比于将支持向量机应用在输入属性$x$上，我们更需要知道如何将其使用在输入特征$\phi(x)$上。其实方法很简单，只需要回顾前面的算法，将式子中的$x$变为$\phi(x)$即可。

前面我们知道了支持向量机算法完全可以使用向量内积$\langle x,z\rangle$表示，所以现在我们可以将这些内积替换为$\left\langle\phi(x),\phi(z)\right\rangle$。特别的，对于给定的特征映射$\phi$，我们定义相应的**核（kernel）**为：

$$K(x,z)=\phi(x)^T\phi(z)$$

如此，我们就可以将前面出现的$\langle x,z\rangle$统统替换为$K(x,z)$，而此时的算法将使用$\phi$作为学习的特征。

对于给定的$\phi$，我们可以很容易的通过求$\phi(x)$与$\phi(z)$的内积得到$K(x,z)$。更有意思的是，即使$\phi$本身很难计算（比如它可能是一个维数极高的向量甚至是无限维向量，这种情况下计算$\phi(x)$的代价将会非常大甚至是不可能的，但是我们可以用很小的代价计算两个向量之间的核），$K(x,z)$也会很容易得到。实际上，通过我们的算法高效解出$K(x,z)$后，就可以使用支持向量机在维数极高的特征空间（特征空间由$\phi$给出）中学习，而且甚至不需要将向量$\phi(x)$计算或表达出来。

我们来看一个例子：设$x,z\in\mathbb{R}^n$，考虑：

$$K(x,z)=\left(x^Tz\right)^2$$

我们可以将其展开为：

$$\begin{align}K(x,z)&=\left(\sum_{i=1}^nx_iz_i\right)\left(\sum_{j=1}^nx_jz_j\right)\\&=\sum_{i=1}^n\sum_{j=1}^nx_ix_jz_iz_j\\&=\sum_{i,j=1}^n\left(x_ix_j\right)\left(z_iz_j\right)\end{align}$$

于是我们可以得知$K(x,z)=\phi(x)^T\phi(z)$中的$\phi$应为（这里我们先假设$n=3$）：

$$\phi(x)=\begin{bmatrix}x_1x_1\\x_1x_2\\x_1x_3\\x_2x_1\\x_2x_2\\x_2x_3\\x_3x_1\\x_3x_2\\x_3x_3\end{bmatrix}$$

我们发现，求出$\phi(x)$需要$O(n^2)$的复杂度，而求出$K(x,z)$仅仅需要$O(n)$的复杂度，也就是与输入属性的维数呈线性关系。

再看另一个核：

$$\begin{align}K(x,z)&=\left(x^Tz+c\right)^2\\&=\sum_{i,j=1}^n\left(x_ix_j\right)\left(z_iz_j\right)+\sum_{i=1}^n\left(\sqrt{2c}x_i\right)\left(\sqrt{2c}z_i\right)+c^2\end{align}$$

这个核相应的特征映射为（仍然假设$n=3$）：

$$\phi(x)=\begin{bmatrix}x_1x_1\\x_1x_2\\x_1x_3\\x_2x_1\\x_2x_2\\x_2x_3\\x_3x_1\\x_3x_2\\x_3x_3\\\sqrt{2c}x_1\\\sqrt{2c}x_2\\\sqrt{2c}x_3\\c\end{bmatrix}$$

式中的参数$c$控制$x_i$（一次项）与$x_ix_j$（二次项）之间的权重。

推广到高维，$K(x,z)=\left(x^Tz+c\right)^d$相应的特征映射在$\binom{n+d}{d}$特征空间中，也就是一共有$\binom{n+d}{d}$个项特征单项式，其中每一项型为$x_{i1}x_{i2}\cdots x_{ik}$一直到$d$阶。尽管在复杂度为$O(n^d)$的高维空间中，对$K(x,z)$的计算仍然仅有$O(n)$的线性复杂度，因为我们并不需要在这种维度极高的空间中直接计算出特征值向量。

我们再来换一个角度观察核，能够直观（这种直观印象并非严格成立）的看出，如果向量$\phi(x)$与$\phi(z)$方向靠的比较近，那么根据$K(x,z)=\phi(x)^T\phi(z)$可知这个值会比较大；反正，如果两个向量近乎正交（方向离的比较远），则$K(x,z)=\phi(x)^T\phi(z)$将会很小。如此，我们就可以用$K(x,z)$度量$\phi(x)$与$\phi(z)$的相似度，或者$x$与$z$的相似度。

基于这种直观印象，如果我们在处理某个学习问题，在遇到$K(x,z)$时，会联想到这个量可以合理的计算$x,z$的相似度。举个例子，介绍我们选择如下的$K$：

$$K(x,z)=\exp\left(-\frac{\left\lVert x-z\right\rVert^2}{2\sigma^2}\right)$$

在这个例子中，$K$可以得到一个合理的关于$x$与$z$的相似度，当$x$和$z$靠近时$K$接近$1$，当$x$和$z$分开时$K$接近$0$。那么，我们可以使用这个$K$作为支持向量机的核吗？在本例中，答案是肯定的。（这个核也叫**高斯核（Gaussian kernel：[中文](https://zh.wikipedia.org/wiki/%E5%BE%84%E5%90%91%E5%9F%BA%E5%87%BD%E6%95%B0%E6%A0%B8)，[英文](https://en.wikipedia.org/wiki/Radial_basis_function_kernel)）**，相应的$\phi$是一个无限维的特征映射）。但是更一般的，对于一个给定的$K$，我们怎么才能知道它是否是一个有效的核？也就是说，对于一个$K$，怎样判断是否存在一个特征映射$\phi$对于所有的$x,z$，能够使得$K(x,z)=\phi(x)^T\phi(z)$？（即判断$\exists\phi\ \mathrm{s.t.}\ K(x,z)=\left\langle\phi(x),\phi(z)\right\rangle$是否成立。）

假设对于某个$\phi$，我们已经有了一个有效核$K$，现在我们有一个有限集合$\left\{x^{(1)},\cdots,x^{(m)}\right\}$（不一定是训练集，可以是任意一个包含$m$个点的集合），含有$m$个点，然后在构造一个$m$阶方阵$K$，它的第$(i,j)$个元素定义为$K_{ij}=K(x^{(i)},x^{(j)})$，我们把这个矩阵成为**核矩阵（kernel matrix）**。注意到我们重复使用了记号$K$，既用来表示核函数$K(x,z)$，又用来表示核矩阵$K$，因为这两个概念很接近。

那么，如果$K$是一个有效的核，则$K_{ij}=K\left(x^{(i)},x^{(j)}\right)=\phi\left(x^{(i)}\right)^T\phi\left(x^{(j)}\right)=\phi\left(x^{(j)}\right)^T\phi\left(x^{(i)}\right)=K\left(x^{(j)},x^{(i)}\right)=K_{ji}$，即$K$为对称矩阵。还有，如果我们使用$\phi_k(x)$表示向量$\phi(x)$的第$k$个元素，则对于任意向量$z$都有：

$$\begin{align}z^TKz&=\sum_{i}\sum_{j}z_iK_{ij}z_j\\&=\sum_i\sum_jz_i\phi\left(x^{(i)}\right)^T\phi\left(x^{(j)}\right)z_j\\&=\sum_i\sum_jz_i\left(\sum_k\phi_k\left(x^{(i)}\right)\phi_k\left(x^{(j)}\right)\right)z_j\\&=\sum_k\sum_i\sum_jz_i\phi_k\left(x^{(i)}\right)\phi_k\left(x^{(j)}\right)z_j\\&=\sum_k\underbrace{\left(\sum_iz_i\phi_k\left(x^{(i)}\right)\right)}_{z^T\phi_k(x)}\underbrace{\left(\sum_jz_j\phi_k\left(x^{(j)}\right)\right)}_{z^T\phi_k(x)}\\&=\sum_k\left(\sum_iz_i\phi_k\left(x^{(i)}\right)\right)^2\\&\geq0\end{align}$$

由于$z$是任意向量，所以$K$是一个半正定矩阵（$K\geq0$）。（倒数第三步的证明可以在[问题集2](http://cs229.stanford.edu/materials/ps2.pdf)问题1中找到思路。）

于是，我们证明了如果$K$是有效的核（即存在相应的特征映射$\phi$），则相应的核矩阵$K\in\mathbb{R}^{m\times m}$是半正定对称矩阵。一般的，可以证明这个条件也是“$K$是一个有效核”的充要条件。接下来的结论基于**Mercer定理（Mercer’s theorem）**。（很多教材通过引入$L^2$函数介绍Mercer定理，略显复杂，因为如果输入属性仅在$\mathbb{R}^n$上取值，与我们接下来给出的结论是等价的。）

**定理（Mercer）**：定义$K:\mathbb{R}^n\times\mathbb{R}^n\to\mathbb{R}^n$，则$K$是有效的（Mercer）核的充要条件是：对于任意$\{x^{(1)},\cdots,x^{(m)}\},\ (m\lt\infty)$，$K$相对应的核矩阵为半正定对称矩阵。

对于给定的函数$K$，除了寻找相应的特征映射$\phi$外，Mercer定理提供了另一种测试核有效性的方法。（在[问题集2](http://cs229.stanford.edu/materials/ps2.pdf)中可以深入理解这个概念。）

为什么我们要将输入属性转变为输入特征，将其从低维转换成高维呢？回忆一下我们为什么要引出支持向量机——我们引出支持向量机是为了解决非线性学习问题，来看下图（来自一篇[介绍核方法的博客](http://blog.csdn.net/xianlingmao/article/details/7719122)）：

<img src="./resource/chapter08_image01.png" width="600" alt="" align=center />

比如说我们的原始数据如左图所示，一个一维的输入属性，而核的作用是将原始输入属性映射到高维特征空间，即右图所示，然后我们在高维特征空间运行支持向量机，就可以找到最优间隔分类器（即分类器将高维空间的的数据分隔开来并使几何间隔最大化）。在本例中，原始数据不是线性可分隔的，当时将它映射在高维空间中，数据就变成线性可分隔的了，于是我们就可以使用线性分类器对原始空间中并非线性可分隔的数据进行分类了。这就是支持向量机输出非线性决策边界的整个过程。（[维基百科关于核方法的解释](https://en.wikipedia.org/wiki/Kernel_method)。）

我们在讲解中还会简要的介绍一些关于其他核的例子。比如，考虑数字识别问题，输入是一张手写的数字（数字$0-9$）图片（$16\times16$像素），我们需要预测图片里的数字到底是几。我们可以使用一个简单多项核$K(x,z)=\left(x^Tz\right)^d$，也可以使用高斯核，而支持向量机将在这种问题上大显身手。令人惊奇的是，因为输入是一个关于图像像素亮度的$256$维向量，而我们的模型对视觉并没有先验知识，它甚至不知道哪些像素是相邻的。

再举一个例子，我们试图分类的对象$x$是一个字符串（假设，它是一个氨基酸列表，连在一起能够组成蛋白质），看上去很难构造一个对于其他学习算法大小较为合理（其他算法通常要求特征值向量不宜过大）的特征值集合，尤其是当不同的字符串长短不一时。令$\phi(x)$为统计$x$中长度为$k$的子字符串出现次数的特征值向量，如果考虑英文字母，则共有$26^k$种子字符串。$\phi(x)$是一个$26^k$维向量，即使再有节制的取$k$，也很难保证我们高效的处理如此高维度的向量。（比如只取$4$位就有$26^4\approx460,000$。）然而使用（动态规划的）字符匹配算法，就可以高效的计算$K(x,z)=\phi(x)^T\phi(x)$，毫无疑问的，现在已经可以处理$26^k$维度的问题了，而且并不需要直接计算出该空间中的特征值向量。

我们已经讨论了将核方法应用于支持向量机的过程，一定要了解的是，核方法的适用性比支持向量机更广泛。特别是当遇到只用输入属性向量的内积$\left\langle x,z\right\rangle$就可以表示的学习算法时，如果我们使用核方法$K(x,z)$代替内积，就会“神奇”的发现现在的算法可以有效的处理$K$相关的维度极高的特征空间了。比如，将核方法用在感知算法上，就可以得到核感知算法。我们在后面将要介绍的很多算法都可以使用核方法处理，这个过程也叫作“核技巧（kernel trick）”。

## 8. 正则化及不可分隔情形

到目前为止，我们都是在数据线性可分隔的假设下讨论支持向量机算法的。通过$\phi$将数据映射到更高纬度的特征空间通常能够增加数据可分隔的概率，但是我们并不能保证总是如此。受可能存在的离群值的影响，在某些情形下，我们并不能直接用算法确定分类超平面。比如下图的情形：

<img src="./resource/chapter08_image02.png" width="600" alt="" align=center />

左图为最优间隔分类器，当我们在左上角加入了一个离群值后，如右图所示，它引起了判别边界剧的烈变动，而新的判别边界的间隔变小了不少。

为了使算法在面对线性不可分隔的数据集时有效，同时也为了降低算法对离群值的敏感度，我们重新定义了如下的优化目标（使用$\mathscr{l}_1$**正则化（regularization）**，也叫$\mathscr{l}_1$ norm soft margin SVM）：

$$\begin{align}\operatorname*{min}_{w,b}&\quad\frac{1}{2}\lVert w\rVert^2+C\sum_{i=1}^m\xi_i\\\mathrm{s.t.}&\quad y^{(i)}\left(w^Tx^{(i)}+b\right)\geq1-\xi_i,\quad i=1,\cdots,m\\&\quad\xi_i\geq0,\quad i=1,\cdots,m\end{align}$$

可以看到，现在我们允许样本的（函数）间隔小于$1$（甚至小于零，函数间隔小于零也就意味着产生了分类错误，我们允许这种情况的发生），而每当样本的函数间隔为$1-\xi_i,\ \xi\gt0$时，我们就给目标函数（总体）加上大小为$C\xi_i$的惩罚开销（也就是说，我们并不鼓励减小间隔，更不鼓励分类错误，但是这是被允许的）。参数$C$用来控制我们的两个目标之间的权重，一是减小$\lVert w\rVert^2$（我们之前看到这样可以增大间隔），二是保证大部分样本的函数间隔至少为$1$。

和以前一样，我们需要重新构造拉格朗日算子：

$$\mathcal{L}(w,b,\xi,\alpha,r)=\frac{1}{2}w^Tw+C\sum_{i=1}^m\xi_i-\sum_{i=1}^m\alpha_i\left[y^{(i)}\left(x^Tw+b\right)-1+\xi_i\right]-\sum_{i=1}^mr_i\xi_i$$

其中$\alpha_i,r_i$是拉格朗日乘数（均为非负实数）。这里我们就不再详细写出对偶问题的推导过程了，在分别计算其关于$w$与$b$的偏导数后将偏导数置为零，再将结果代回拉格朗日算子并化简，得到如下对偶问题：

$$\begin{align}\displaystyle\operatorname*{max}_{\alpha}&\quad W(\alpha)=\sum_{i=1}^m\alpha_i-\frac{1}{2}\sum_{i,j=1}^my^{(i)}y^{(j)}\alpha_i\alpha_j\left\langle x^{(i)},x^{(j)}\right\rangle\\\mathrm{s.t.}&\quad0\leq\alpha_i\leq C,\quad i=1,\cdots,m\\&\quad\sum_{i=1}^m\alpha_iy^{(i)}=0\end{align}$$

参见上一讲的$(9)$式，我们可以将$w$写为关于$\alpha$的函数，在计算出对偶问题的解之后，可以使用上一讲的$(13)$式对新样本进行预测。有趣的是，加入了$\mathscr{l}_1$正则化后，唯一的改变就是约束条件从原来的$0\leq\alpha_i$变为现在的$0\leq\alpha_i\leq C$。$b^*$的计算也发生了改变（上一讲的$(11)$式不再有效，参考下一节的讲解及Platt的文章）。

由原来的KKT对偶互补条件推导出的新的KKT对偶互补条件（在下一节中也会用于测试序列最小优化算法的收敛情况）：

$$\begin{align}\alpha_i=0&\implies y^{(i)}\left(w^Tx^{(i)}+b\right)\geq1\tag{14}\\\alpha_i=C&\implies y^{(i)}\left(w^Tx^{(i)}+b\right)\leq1\tag{15}\\0\lt\alpha_i\lt C&\implies y^{(i)}\left(w^Tx^{(i)}+b\right)=1\tag{16}\end{align}$$

万事俱备，接下来我们就缺一个能够直接求解对偶问题的算法了，下一节就来介绍这个算法。

## 9. 序列最小优化算法（SMO:  sequential minimal optimization）

序列最小优化算法来源于John Platt，他找到了一种高效解决“由支持向量机引出的对偶问题”的算法。在继续介绍该算法之前，我们先来看一下坐标下降/上升算法。

### 9.1 坐标下降/上升法

考虑如何解决一个不加约束条件的优化问题：

$$\operatorname*{max}_\alpha W\left(\alpha_1,\alpha_2,\cdots,\alpha_m\right)$$

现在，我们只将$W$看做关于$\alpha_i$的函数，与支持向量机无关。到目前为止我们已经了解了两种优化算法：梯度下降/上升法，牛顿法，这里我们再来介绍一种新的优化算法——**坐标下降/上升法（coordinate descent/ascent algorithm，[中文](https://zh.wikipedia.org/wiki/%E5%9D%90%E6%A0%87%E4%B8%8B%E9%99%8D%E6%B3%95)，[英文](https://en.wikipedia.org/wiki/Coordinate_descent)）**：

```
Loop until convergence {
    for i=1 to m, {
```
$\quad\qquad\quad\quad\displaystyle\alpha_i:=\mathrm{arg}\operatorname*{max}_{\hat\alpha_i}W\left(\alpha_1,\cdots,\alpha_{i-1},\hat\alpha_i,\alpha_{i+1},\cdots,\alpha_m\right)$
```
    }
}
```

在算法里面一层循环中，我们会固定除$\alpha_i$以外的参数，然后重新优化这个关于$\alpha_i$的函数$W$。对上面的这个例子，里面的循环会按照$\alpha_1,\alpha_2,\cdots,\alpha_m,\alpha_1,\alpha_2,\cdots$的顺序进行优化。（对一些复杂的问题可能会使用别的顺序，比如，我们可以看哪个参数会带来$W(\alpha)$最大幅度的增长，就选择哪个参数作为下一个优化的对象。）

而当函数$W$恰好符合里面一层循环可以高效运行的状态时（事实上有很多优化问题很容易固定其他参数只对一个参数求最优值，在这种情况下本算法的内层循环将会执行的非常快。而支持向量机通常也属于这种情况），坐标下降/上升法确实是一个效率很高的算法，如图所示：

<img src="./resource/chapter08_image03.png" width="400" alt="" align=center />

这是一个需要优化的二次函数的等高线图，坐标上升算法初始值取在$(2,-2)$，图中的线就是坐标上升算法从初始值到全局最大值的优化路径。注意到算法的每一步都是沿着坐标轴方向优化，因为算法每次只优化一个参数。

### 9.2 序列最小优化

我们通过简要的介绍序列最小优化来完结支持向量机算法，推导过程中将有一些细节被省略。

我们再一次列出（对偶）优化问题：

$$\begin{align}\displaystyle\operatorname*{max}_{\alpha}&\quad W(\alpha)=\sum_{i=1}^m\alpha_i-\frac{1}{2}\sum_{i,j=1}^my^{(i)}y^{(j)}\alpha_i\alpha_j\left\langle x^{(i)},x^{(j)}\right\rangle\tag{17}\\\mathrm{s.t.}&\quad0\leq\alpha_i\leq C,\quad i=1,\cdots,m\tag{18}\\&\quad\sum_{i=1}^m\alpha_iy^{(i)}=0\tag{19}\end{align}$$

假设我们已经有一组满足约束条件$(18-19)$式的$\alpha_i$，现在，我们固定$\alpha_2,\cdots,\alpha_m$，然后进行坐标上升优化步骤，重新优化关于$\alpha_1$的目标函数，那么我们能够得到更加优化的目标函数吗？答案是否定的，因为$(19)$式：

$$\alpha_1y^{(1)}=-\sum_{i=2}^m\alpha_iy^{(i)}$$

或者上式两边同时乘以$y^{(1)}$有：

$$\alpha_1=-y^{(1)}\sum_{i=2}^m\alpha_iy^{(i)}$$

（因为$y^{(1)}\in\{-1,1\}$，所以$\left(y^{(1)}\right)^2=1$。）我们发现，$\alpha_1$直接可以由其余的$\alpha_i$确定，所以如果我们固定$\alpha_2,\cdots,\alpha_m$，在不违反约束条件$(19)$的前提下，我们根本不能改变$\alpha_1$。

如果我们想要更新某个$\alpha_i$，我们就至少得同时更新两个参数以保证算法仍在约束条件内。这就是序列最小优化算法（最小就是指一次改变最少个$\alpha_i$，在这里即两个）的思路：

Repeat till convergence {
1. 选择一对参数$\alpha_i,\alpha_j$作为将要优化的对象（启发式（也就是经验法则）的找到两个可以使优化目标朝全局最大值最快收敛的参数）。
2. 优化关于$\alpha_i,\alpha_j$的函数$W(\alpha)$，也就是将其余的$\alpha_k,k\neq i,j$。

}

为了测试算法的收敛性，我们可以检测算法是否满足带有某$tol$的KKT条件（即$(14-16)$式）。其中$tol$是收敛容差参数（convergence tolerance parameter），通常设为$0.01$到$0.001$。

序列最小优化是一个高效算法的关键在于对$\alpha_i,\alpha_i$的计算效率非常高（它可能比牛顿法迭代的步骤更多，但是它每次迭代的代价非常小）。我们来简要推导一下这种高效的参数更新。

假设我们有一组满足约束条件$(18-19)$的$\alpha_i$，现在将参数$\alpha_3,\cdots,\alpha_m$固定，然后优化关于$\alpha_1,\alpha_2$的函数$W(\alpha_1,\alpha_2,\cdots,\alpha_m)$（在约束条件下）。根据$(19)$式得到：

$$\alpha_1y^{(1)}+\alpha^{(2)}=-\sum_{i=3}^m\alpha_iy^{(i)}$$

等式右边是固定的（因为我们固定了参数$\alpha_3,\cdots,\alpha_m$），我们把右边记作常量$\zeta$：

$$\alpha_1y^{(1)}+\alpha^{(2)}=\zeta\tag{20}$$

我们可以用下图表示$\alpha_1,\alpha_2$上的约束条件：

<img src="./resource/chapter08_image04.png" width="400" alt="" align=center />

从$(18)$得知$\alpha_1,\alpha_2$必须落在图中$[0,C]\times[0,C]$的区域内，图中的斜线为$\alpha_1y^{(1)}+\alpha^{(2)}=\zeta$，即$\alpha_1,\alpha_2$所在直线。从约束条件我们还能得到$L\leq\alpha_2\leq H$；否则$(\alpha_1,\alpha_2)$就不能同时满足图中的区域约束和直线约束了。在本例中$L=0$。但并不总是如此，这需要参考直线$\alpha_1y^{(1)}+\alpha^{(2)}=\zeta$。通常，$\alpha_2$可能会有更低的下限$L$和更高的上限$H$，但总会保证$\alpha_1,\alpha_2$在$[0,C]\times[0,C]$的区域内。

通过$(20)$式，我们可以将$\alpha_1$表示成关于$\alpha_2$的函数：

$$\alpha_1=\left(\zeta-\alpha_2y^{(i)}\right)y^{(1)}$$

（我们又一次使用了由$y^{(1)}\in\{-1,1\}$推出$\left(y^{(1)}\right)^2=1$的技巧。）因此$W(\alpha)$可以写为：

$$W(\alpha_1,\alpha_2,\cdots,\alpha_m)=W\left(\left(\zeta-\alpha^{(2)}y^{(2)}\right)y^{(1)},\alpha_2,\cdots,\alpha_m\right)$$

将$\alpha_3,\cdots,\alpha_m$当做常数，易知这就是一个关于$\alpha_2$的二次函数。也就是说可以将$W$看做$a\alpha_2^2+b\alpha_2+c$（选用合适的$a,b,c$作为系数）。如果忽略$(18)$式的区域约束条件（或等价的$L\leq\alpha_2\leq H$），则我们可以通过将这个二次函数的导数置为零的方法，求出最大值。我们使用$\alpha_2^{new,unclipped}$标记解出的$\alpha_2$。如果我们服从$(18)$式的约束条件，想要最大化$W$关于$\alpha_2$的函数时，只需要简单的将$\alpha_2^{new,unclipped}$“剪入”$[L,H]$的区间即可：

$$\alpha_2^{new}=\begin{cases}H&if\ \alpha_2^{new,unclipped}\gt H\\\alpha_2^{new,unclipped}&if\ L\leq\alpha_2^{new,unclipped}\leq H\\L&if\ \alpha_2^{new,unclipped}\lt L\end{cases}$$

当我们求出$\alpha_2^{new}$之后，代回$(20)$式即可得到$\alpha_1^{new}$的优化值。

这个算法还有一些别的细节，可以参考Platt的文章：“ the choice of the heuristics used to select the next $\alpha_i$, $\alpha_j$ to update”和“how to update $b$ as the SMO algorithm is run”。