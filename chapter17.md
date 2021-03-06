
# 第十七讲：解连续状态的MDP

在上一讲，我们给出了强化学习和马尔可夫决策过程的定义。

MDP是一个五元组$\left(S,A,\{P_{sa}\},\gamma,R\right)$，上一讲我们一直使用的一个低阶MDP的例子——机器人躲避障碍物到达目的地（$+1$）：

$$
\begin{array}
{|c|c|c|c|}
\hline
?&?&?&+1\\
\hline
?&\textrm{墙}&?&-1\\
\hline
?&?&?&?\\
\hline
\end{array}
$$

* 在这个例子中，共有$11$个状态——$\lvert S\rvert=11$；
* 机器人能够做出的动作为$A=\{\textrm{N, S, E, W}\}$，即向北、南、东、西移动；
* 状态转换概率$P_{sa}$为处于状态$s$时，做出动作$a$后状态变为$s'$的概率，比如从状态$(3,1)$开始，做出动作“向北移动”，则机器人模型可能会因为机械传动噪音有$0.1$的概率“向东移动”、有$0.1$的概率的概率“向西移动”、有$0.8$的概率“向北移动”，则真正落在状态$(3.2)$的概率为$0.8$如图：

$$
\begin{array}
{|c|c|c|c|}
\hline
?&?&?&+1\\
\hline
?&\textrm{墙}&0.8\uparrow&-1\\
\hline
?&0.1\leftarrow&s&0.1\rightarrow\\
\hline
\end{array}
$$

* 奖励函数$R=\begin{cases}\pm1&\textrm{absorbing state}\\-0.02&\textrm{else where}\end{cases}$，即在$(4,2)$时奖励为$-1$，代表任务失败并结束；而在$(4,3)$状态代表奖励$+1$，代表任务成功并结束；其他状态奖励$-0.02$（**吸收状态（absorbing state）**表示进入该状态即时奖励$+1$，并且其他状态奖励均归零，也就是任务结束）。

* 折扣因子$\gamma=0.99$，这个值代表当前奖励与未来奖励间的权重（当前为$\gamma^0$，下一步为$\gamma^1$，再下一步为$\gamma^2$等等，这一项鼓励模型尽早获得奖励）。

而策略$\pi:S\to A$，对本例来说就是一个机器人控制策略，它是一个从状态到动作的映射，也就是告诉机器人在每一种状态下该执行什么动作。我们的目标就是找到一个能够最大化我们预期总收益的策略（即尽可能多的获得奖励）。

我们定义了策略$\pi$的价值函数$V^\pi(s)=\mathrm E\left[R(s_0)+\gamma R(s_1)+\gamma^2R(s_2)+\cdots\right]$，解释为“在状态$s$下启动机器人并使其在策略$\pi$下运行，所得到的折扣奖励之和的期望”。对于目标而言，其中一种方法是找到最大的价值函数：

1. 我们先要计算$\displaystyle V^*(s)=\max_\pi V^\pi(s)$——这是执行任意策略能够得到的最大收益，比如这个最优价值函数看上去如下图所示：
    $$
    \begin{array}
    {|c|c|c|c|}
    \hline
    .86&.90&.95&+1\\
    \hline
    .82&\textrm{墙}&.69&-1\\
    \hline
    .78&.75&.71&.49\\
    \hline
    \end{array}
    $$
    这就是$V^*$，它表示从任意状态开始，能够得到的折扣奖励之和的预期值。

2. 一旦得到了$V^*$，我们就可以使用$\displaystyle\pi^*(s)=\arg\max_{a\in A}\sum_{s'\in S}P_{sa}\left(s'\right)V^*\left(s'\right)$计算出最佳策略$\pi^*$。
    $$
    \begin{array}
    {|c|c|c|c|}
    \hline
    \rightarrow&\rightarrow&\rightarrow&+1\\
    \hline
    \uparrow&\textrm{墙}&\uparrow&-1\\
    \hline
    \uparrow&\leftarrow&\leftarrow&\leftarrow\\
    \hline
    \end{array}
    $$
3. 算法的实现就是利用贝尔曼方程$\displaystyle V^*(s)=R(s)+\max_{a\in A}\gamma\sum_{s'\in S}P_{s\pi(s)}\left(s'\right)V^*\left(s'\right)$，在状态$s$时的最优折扣奖励之和为“在$s$开始时的即时奖励”加上“所有动作中能够最大化未来折扣奖励总和动作序列对应的未来奖励”。于是我们可以应用一个值迭代算法，也就是利用贝尔曼方程不停的迭代$\displaystyle V(s):=R(s)+\max_{a\in A}\gamma\sum_{s'\in S}P_{sa}\left(s'\right)V\left(s'\right)$。这样$V(s)$就会收敛于$V^*(s)$，而有了$V^*(s)$就可以得到$\pi^*(s)$。

最后，我们还介绍了策略迭代算法：根据随机初始化的策略$\pi$计算$V^\pi$并更新$V$（即为特定的策略计算出价值函数，使用贝尔曼方程组计算每个状态的折扣奖励总和），然后假设我们已经找到的策略是最佳策略，根据$\displaystyle\pi(s):=\arg\max_{a\in A}\sum_{s'\in S}P_{sa}\left(s'\right)V\left(s'\right)$带入假设的最优价值函数$V$找到新的策略再更新$\pi$。重复这两步最终会使得$V$收敛于$V^*$，$\pi$收敛于$\pi^*$。如图：

1. 随机初始化一个策略：
    $$
    \begin{array}
    {|c|c|c|c|}
    \hline
    \downarrow&\rightarrow&\rightarrow&+1\\
    \hline
    \rightarrow&\textrm{墙}&\rightarrow&-1\\
    \hline
    \uparrow&\rightarrow&\uparrow&\leftarrow\\
    \hline
    \end{array}
    $$
2. 解贝尔曼方程组$\displaystyle V^\pi(s)=R(s)+\gamma\sum_{s'\in S}P_{s\pi(s)}\left(s'\right)V^\pi\left(s'\right)$，在本例中有$11$种状态，所以就有由$11$个价值函数组成的线性方程组，解这个方程组就可以得到个状态对应的价值函数的值，再填在图中即可得到一个新的策略。

以上就是上一讲的主要内容，下面我们会接着这两种算法（值迭代与策略迭代）继续MDP的介绍。

## 4. 具有连续状态的MDP

到目前为止，我们介绍的都是具有有限个状态的MDP。接下来，我们要讨论具有无限个状态的MDP。举个例子，对于一辆车，我们会用状态$(x,y,\theta,\dot x,\dot y,\dot\theta)$来表示，其中$(x,y)$表示车子的位置；$\theta$表示车子的方向；车子在$x,y$方向上的速度分量$\dot x,\dot y$；以及角速度$\dot\theta$。因此，$S=\mathbb R^6$是一个无限状态集，因为对于一辆车来说，有无限个可能的位置和方向。（理论上讲，车子的方向$\theta$应该在$\theta\in[-\pi,\pi)$取值，而不是$\theta\in\mathbb R$，但在我们的例子中，这一点并不重要。）类似的，在[问题集4](http://cs229.stanford.edu/materials/ps4.pdf)中的倒置钟摆问题，钟摆的状态为$(x,\theta,\dot x,\dot\theta)$，而$\theta$就是钟摆的角度。再比如我们的遥控直升机，在三维空间中飞行，它的状态为$(x,y,z,\phi,\theta,\psi,\dot x,\dot y,\dot z,\dot\phi,\dot\theta,\dot\psi)$，其中$\phi$为横滚角、$\theta$为俯仰角、$\psi$为航向角。

在本节我们将考虑状态空间为$S=\mathbb R^n$的情形，并介绍如何解出这种MDP。
### 4.1 离散化（Discretization）

描述一个连续状态MDP的最简单的方法就是将它的状态空间离散化，然后在对其应用前面介绍的值迭代或策略迭代进行求解。

比如我们有一个二维状态$(s_1,s_2)$，我们就可以使用网格将其状态空间离散化：

<img src="./resource/chapter17_image01.png" width="300" alt="" align=center />

图中的每一格都代表一个独立的状态$\bar s$。于是，我们就可以使用一个离散状态的MDP$\left(\bar S,A,\left\{P_{\bar sa}\right\},\gamma,R\right)$去近似原来的连续状态MDP，而$\left\{P_{\bar sa}\right\}$是关于离散状态的状态转换概率。然后就可以应用前面介绍过的值迭代或策略迭代解出MDP$\left(\bar S,A,\left\{P_{\bar sa}\right\},\gamma,R\right)$的$V^*\left(\bar s\right)$和$\pi^*\left(\bar s\right)$。当我们的实际系统处于状态$s\in S$时，我们需要选择一个动作并执行，此时我们就应该计算连续状态$s$相对应的离散状态$\bar s$，然后执行动作$\pi^*(\bar s)$。

这种基于离散化的实现方法可以应用与很多MDP问题的解决，但是它有两个缺点：

* 首先是该方法对于$V^*$（以及$\pi^*$）的表达过于纯粹，方法假设价值函数在每一个离散化的区间上取常数值（即价值函数对于每个网格来说是一个分段常数）。

    为了更加形象化的理解这种表达的局限，考虑监督学习问题中使用函数拟合数据集的过程：

    <img src="./resource/chapter17_image02.png" width="400" alt="" align=center />

    很容易看出，做线性回归就可以很好的拟合数据。然而如果我们将$x$坐标离散化，然后使用分段常数来表示离散区间，则对数据的拟合将变成下图中的样子：

    <img src="./resource/chapter17_image03.png" width="400" alt="" align=center />

    相比于平滑的函数，这种分段常数表示法有一些缺陷。对于输入来说它得到的输出不够平滑，而且无法将不同的离散区间归纳在同一个表达式中。对于这种数据表示法，我们需要一个很细的离散化（网格非常细）才能做出较好的近似。

* 另一个缺陷则是**维数灾难（curse of dimensionality）**。设$S=\mathbb R^n$，我们将$n$维中的每一个维度离散化成$k$个值，那么离散的状态的总数就有$k^n$个之多。对于状态空间的维数$n$来说，这个离散状态总数呈指数增加，所以并不适用于大型问题的解决。对于一个$10$维状态空间，如果我们将每个维度离散化为$100$个值，那么就会得到$100^{10}=10^{20}$个离散状态，这对目前的计算机来说太大了。

    按照实践经验，离散化在解决$1$维或$2$为问题时效果非常好（且实现起来方便快捷）。也许，凭借一些智慧并在离散化方法选择上小心谨慎，那么这种方法也能够处理$4$维问题。如果我们特别智慧又非常幸运的话，可能能够让这种方法撑到解决$6$维问题。但是如果问题复杂度再增加，这种方法就无能为力了。

### 4.2 价值函数近似

我们再来介绍另一种能够求解具有连续状态的MDP的方法，这次我们直接对$V^*$做近似，也就省去了离散化的步骤。这种实现称为**价值函数近似（value function approximation）**，已经应用在很多强化学习问题中了。

#### 4.2.1 使用模型或模拟器

为了实现价值函数近似，我们需要为MDP假设一个**模型（model）**或**模拟器（simulator）**。非正式的定义，模拟器是一个黑盒，能够接受任意的（连续值）状态$s_t$和动作$a_t$作为输入，并按照状态转换概率$P_{s_ta_t}$输出下一个状态$s_{t+1}$：

<img src="./resource/chapter17_image04.png" width="400" alt="" align=center />

我们有几种得到模型的方法，其中一种就是物理模拟。比如[问题集4](cs229.stanford.edu/materials/ps4.pdf)中倒置钟摆的例子，我们假设系统的所有参数已知，比如杆的长度、杆的质量等等，根据这些输入参数，使用各种物理定律计算位于$t$时刻的当前状态在执行动作$a$之后，系统在$t+1$时刻系统的状态（包括小车的、杆的角度等）。我们也可以使用现成的物理模拟软件，我们只需要在软件中对这个机械系统进行物理建模，并给出当前状态$s_t$和要执行的动作$a_t$，就可以计模拟出未来一小段时间内，处于$t+1$时刻系统的状态。（比如[Open Dynamics Engine](http://www.ode.com)就是一款免费的开源物理模拟器，它完全可以用来模拟“倒置钟摆”问题，并已经在强化学习研究者中建立了良好的口碑。）

另一种方法是从MDP收集到的数据中“学到”一个模型。举个例子，假设我们通过不停的执行动作，在MDP中做了$m$次**试验（trials）**，每一次试验都进行了$T$步。这样的试验可以通过随机选择动作、依照特定的策略或使用其他选择动作的方式。于是我们可以得到$m$个如下的状态序列：

$$
\require{AMScd}
\begin{CD}
s_0^{(1)} @>{a_0^{(1)}}>> s_1^{(1)} @>{a_1^{(1)}}>> s_2^{(1)} @>{a_2^{(1)}}>> \cdots @>{a_{T-1}^{(1)}}>> s_T^{(1)}
\end{CD}\\
\begin{CD}
s_0^{(2)} @>{a_0^{(2)}}>> s_1^{(2)} @>{a_1^{(2)}}>> s_2^{(2)} @>{a_2^{(2)}}>> \cdots @>{a_{T-1}^{(2)}}>> s_T^{(2)}
\end{CD}\\
\vdots\\
\begin{CD}
s_0^{(m)} @>{a_0^{(m)}}>> s_1^{(m)} @>{a_1^{(m)}}>> s_2^{(m)} @>{a_2^{(m)}}>> \cdots @>{a_{T-1}^{(m)}}>> s_T^{(m)}
\end{CD}\\
$$

接下来，我们就可以应用一种学习算法，类似一个函数，输入是$s_t$和$a_t$，而输出是预测的状态$s_{t+1}$。

比如我们可能会建立一个线性模型：

$$
s_{t+1}=As_t+Ba_t\tag{5}
$$

使用类似于线性回归的算法。此处模型的参数是矩阵$A$和$B$，我们可以使用收集到的$m$次测试数据来估计这两个矩阵：

$$
\arg\min_{A,B}\sum_{i=1}^m\sum_{t=0}^{T-1}\left\lVert s_{t+1}^{(i)}-\left(As_t^{(i)}+Ba_t^{(i)}\right)\right\rVert^2
$$

（也就是求参数的最大似然估计的步骤。）

在算法找出$A,B$之后，其中一种选择是建立一个**确定性（deterministic）**模型，它接受$s_t,a_t$作为输入，并输出一个确定的$s_{t+1}$。比如我们总是使用$(5)$式计算$s_{t+1}$。另一种选择是建立一个**随机性（stochastic）**模型，在这种模型中，$s_{t+1}$是一个关于输入$s_t,a_t$的随机函数：$\displaystyle s_{t+1}=As_t+Ba_t+\epsilon_t$，而$\epsilon_t$是一个噪音项，我们通常令其服从$\epsilon_t\sim\mathcal N(0,\varSigma)$。（协方差矩阵$\varSigma$也可以通过测试数据直接估计出来。）

这里举出的例子是将$s_{t+1}$看做是关于当前状态和动作的线性函数，我们当然也可以使用非线性函数对其建模。比如说用函数$s_{t+1}=A\phi_s(s_t)+B\phi_a(a_t)$建模，其中$\phi_s,\phi_a$是关于状态和动作的某些非线性特征映射。或者，我们也可以使用一些非线性学习算法——比如局部加权回归（参见[第三讲](chapter02.ipynb)）——将$s_{t+1}$拟合为关于$s_t,a_t$的函数。这些实现的方法既可以用来构建确定性模拟器，也可以用来构建随机性模拟器。

#### 4.2.2 拟合值迭代（Fitted value iteration）

现在来介绍**拟合值迭代（fitted value iteration）**算法，用于在具有连续状态的MDP上近似值函数。在后面的讨论中我们假设MDP具有连续的状态空间$S=\mathbb R^n$，但是动作空间$A$规模较小并且是离散的（因为在实践中，通常状态空间的维度要比动作空间大很多，所以动作空间通常比较容易离散化）。

回忆在值迭代中的更新规则：

$$
\begin{align}
V(s)&:=R(s)+\gamma R\int_{s'}P_{sa}\left(s'\right)V\left(s'\right)\mathrm ds'\tag{6}\\
&=R(s)+\gamma\max_a\mathrm E_{s'\sim P_{sa}}\left[V\left(s'\right)\right]\tag{7}
\end{align}
$$

（在[上一讲](chapter16.ipynb)第二节中，我们得到的值迭代更新规则$\displaystyle V(s):=R(s)+\max_{a\in A}\gamma\sum_{s'\in S}P_{sa}\left(s'\right)V\left(s'\right)$中对状态使用了求和运算，而在这里我们对状态使用的是积分运算，这说明我们现在是在连续状态下（而不是在离散状态下）求解MDP。）

拟合值迭代的核心思想就是在$s^{(1)},\cdots,s^{(m)}$的有限抽样上进行近似的迭代步骤。我们将使用监督学习算法（在下面的讨论中将使用线性回归），将值函数看做一个关于状态的线性或非线性函数，然后再来近似这个函数：

$$
V(s)=\theta^T\phi(s)
$$

这里的$\phi$是某个关于状态的映射。

在$m$个状态的有限样本中，对于每一个状态$s$，算法将先计算一个记为$y^{(i)}$的量（我们在后面会把这个量当做$R(s)+\gamma\max_a\mathrm E_{s'\sim P_{sa}}\left[V\left(s'\right)\right]$的近似值，即$(7)$式）；然后会应用一个监督学习算法，尝试使$V(s)$靠近$R(s)+\gamma\max_a\mathrm E_{s'\sim P_{sa}}\left[V\left(s'\right)\right]$（换句话说，尝试使$V\left(s^{(i)}\right)$尽量靠近$y^{(i)}$）。算法的步骤如下：

重复：`{`
1. 随机抽样$m$个状态$s^{(1)},s^{(2)},\cdots,s^{(m)}\in S$；
2. 初始化$\theta:=0$；
3. 对每个状态 $i=1,\cdots,m$ `{`
  * 对每个动作$a\in A$ `{`
    * 抽样$s_1',\cdots,s_k'\sim P_{s^{(i)}a}$（使用MDP模型根据$s,a$预测$s'$）；
    * 令$\displaystyle q(a)=\frac{1}{k}\sum_{j=1}^kR\left(s^{(i)}\right)+\gamma V\left(s_j'\right)$ // 因此，$q(a)$就是一个对$\displaystyle R(s^{(i)})+\gamma\mathrm E_{s'\sim P_{s^{(i)}a}}\left[V\left(s'\right)\right]$的估计；

    `}`
    
  * 令$\displaystyle y^{(i)}=\max_aq(a)$ // 因此，$y^{(i)}$就是一个对$\displaystyle R(s^{(i)})+\gamma\max_a\mathrm E_{s'\sim P_{s^{(i)}a}}\left[V\left(s'\right)\right]$的估计；
  
  `}`
  
  // 在原来的（用于解离散状态的）值迭代算法中，我们按照$V\left(s^{(i)}\right):=y^{(i)}$更新值函数；
  
  // 而在这个算法中，我们将使用监督学习算法（线性回归）来实现$V\left(s^{(i)}\right)\approx y^{(i)}$，即使$V\left(s^{(i)}\right)$尽量靠近$y^{(i)}$；
  * 令$\displaystyle\theta:=\arg\min_\theta\frac{1}{2}\sum_{i=1}^m\left(\theta^T\phi\left(s^{(i)}\right)-y^{(i)}\right)^2$

`}`

以上就是使用线性回归使$V\left(s^{(i)}\right)$靠近$y^{(i)}$的拟合值迭代。类比标准监督学习（回归）算法，在回归算法中我们已知训练集$\left(x^{(1)},y^{(1)}\right),\left(x^{(2)},y^{(2)}\right),\cdots,\left(x^{(m)},y^{(m)}\right)$，想要学习到一个从$x$到$y$的函数映射，这个算法与回归唯一不同的是$s$代替了$y$的位置。虽然上面的例子中使用的是线性回归，不过显然也可以使用其他回归算法（如局部加权回归等）。

与离散状态上的值迭代不同的是，不能被证明收敛。不过不用担心，在实践中该算法通常收敛（或近达到似收敛的程度），而且对很多问题都非常有效。注意到如果我们使用MDP的确定性模拟器/模型，则就可用$k=1$简化算法。这是因为$(7)$式中的期望变成了关于一个确定性分布的期望（在确定性模型中，我们在给定输入下做的每次抽样将得到同样的输出），于是使用一个样本就可以计算相应的期望值了。如果不是确定性模型，那么我们必须抽取$k$个样本，然后再用均值去近似相应的期望（见算法伪代码中关于$q(a)$的定义）。

最后，拟合值迭代的输出$V$是一个关于$V^*$的近似，这同时暗示了策略的定义。当系统处于状态$s$时，需要选择一个动作，于是我们使用：

$$
\arg\max_a\mathrm E_{s'\sim P_{sa}}\left[V^*\left(s'\right)\right]\tag{8}
$$

（由于状态是连续的，我们就无法计算每一个状态的最优价值函数了。所以我们每次在需要做下一个动作时进行计算：在系统处于特定状态时，执行某动作会进入下一个状态$s'$，式子会找到能够使$\displaystyle\mathrm E_{s'\sim P_{sa}}\left[V^*\left(s'\right)\right]$取到最大的动作。）

这一步计算/近似类似于拟合值迭代里层的循环中用抽样$s_1',\cdots,s_k'\sim P_{sa}$近似期望的步骤（同样的，如果模拟器是确定性的，则取$k=1$即可）。

在实践中，我们也经常使用其他方法来近似这个步骤。举个例子：
* 如果我们正在使用一个确定性模型，则有$s_{t+1}=f(s_t,a_t)$（即有了关于当前状态和动作的函数）。于是上面的式子就可以直接简化为$\displaystyle\arg\max_aV^*\left(f(s,a)\right)$（因为$s'=f(s,a)$）；
* 另一种常见的情形是在随机性模型中，当模拟器采用$s_{t+1}=f(s_t,a_t)+\epsilon_t$，其中$f$是关于状态的确定性函数（比如$f(s_t,a_t)=As_t+Ba_t$），$\epsilon$是一个服从零期望高斯分布的噪音项。在这种情况下，我们可以通过

$$
\arg\max_aV^*\left(f(s,a)\right)
$$

选择动作。也就是说令$\epsilon=1$（即忽略模拟器的噪音）并令$k=1$。等价的，这也可以从$(8)$式中利用近似推导出来：

$$
\begin{align}
\mathrm E_{s'}\left[V\left(s'\right)\right]&\approx V\left(\mathrm E_{s'}\left[s'\right]\right)\tag{9}\\
&=V\left(f(s,a)\right)\tag{10}
\end{align}
$$

此处的预期值是在随机的$s'\sim P_{sa}$上的。只要噪音项$\epsilon$较小，则这个近似通常都是合理的。

不过，对于不适合做这种近似的问题，使用模型抽样$k\lvert A\rvert$个状态以近似上面的期望，可能会使计算的代价非常大。
