
# 第十九讲：微分动态规划及线性二次型高斯

在继续学习之前，我们先来回顾一下[上一讲](chapter18.ipynb)的内容：
* 其目标是$\displaystyle\max_a\mathrm E\left[R^{(0)}(s_0,a_0)+R^{(1)}(s_1,a_1)+\cdots+R^{(T)}(s_T,a_T)\right]$；
* 之后我们应用了动态规划算法：
    $$
    \begin{align}
    V_T^*(s)&=\max_{a_T}R(s_T,a_T)\tag{1}\\
    V_t^*(s)&=\max_a\left(R(s,a)+\sum_{s'}P_{sa}^{(t)}\left(s'\right)V_{t+1}^*\left(s'\right)\right)\tag{2}\\
    \pi_t^*(s)&=\arg\max_a\left(R(s,a)+\sum_{s'}P_{sa}^{(t)}\left(s'\right)V_{t+1}^*\left(s'\right)\right)\tag{3}
    \end{align}
    $$
    其中$(1)$式是有限时域MDP的最后一步；得出最后一步后就可以使用$(2)$式向后递推每一个$V_{T-1},\cdots,V_{0}$；最后，在使用$\arg\max$对每一步价值函数最大化的部分做运算，就可以得到每一步的最优策略。
* 接下来我们介绍了一个具体的LQR问题：
    * 状态为$S\in\mathbb R^n$；
    * 动作为$A\in\mathbb R^d$；
    * 将下一步作为关于当前状态及动作的函数$s_{t+1}=A_ts_t+B_ta_t+w_t$，此处的$w_t\sim\mathcal N(0,\varSigma_w)$是一个期望为零、协方差为$\varSigma_w$的高斯噪音项。
* 上面定义中有了关于当前状态及动作的函数$s_{t+1}=f(s_t,a_t)$，我们就可以选择一个系统长时间保持的状态（系统通常都会都处于这种状态）$(\bar s_t,\bar a_t)$，在该点使用线性函数近似非线性函数$s_{t+1}=f(s_t,a_t)$，进而得到$\displaystyle s_{t+1}\approx f(\bar s_t,\bar a_t)+(\nabla_sf(\bar s_t,\bar a_t))^T(s_t-\bar s_t)+(\nabla_af(\bar s_t,\bar a_t))^T(a_t-\bar a_t)$。最终能够在$(\bar s_t,\bar a_t)$附近得到一个线性函数$s_{t+1}=A_ts_t+B_ta_t$。
* LQR的奖励函数为$\displaystyle R^{(t)}(s_t,a_t)=-\left(s_t^TU_ts_t+a^TV_ta_t\right)$，其中矩阵$U,V$是半正定的，所以这个奖励函数总是非正数；
* 对LQR使用上面的动态规划得到求解方法：
    * 使用$\varPhi_t=-U_t,\varPsi_t=0$初始化$V_T^*$；
    * 向后递推计算$\varPhi_t,\varPsi_t$，使用离散时间Riccati方程，将其当做关于$\varPhi_{t+1},\varPsi_{t+1}$的线性函数；（依次求出$t=T-1,T-2,\cdots,0$。）
        $$
        \begin{align}
        \varPhi_t&=A_t^T\left(\varPhi_{t+1}-\varPhi_{t+1}B_t\left(B_t^T\varPhi_{t+1}B_t-V_t\right)^{-1}B_t\varPhi_{t+1}\right)A_t-U_t\\
        \varPsi_t&=-\mathrm{tr}\varSigma_w\varPhi_{t+1}+\varPsi_{t+1}
        \end{align}
        $$
    * 计算$\displaystyle L_t=\left(B_t^T\varPhi_{t+1}B_t-V_t\right)^{-1}B_t^T\varPhi_{t+1}A_t$，这是一个关于$\varPhi_{t+1},\varPsi_{t+1}$的函数；
    * 计算$\pi^*(s_t)=L_ts_t$，这是一个关于$s_t$的线性函数，系数是$L_t$。
    
  这个算法中有趣的一点是，$L_t$不依靠$\varPsi_t$，$\varPhi_t$也不依赖于$\varPsi_t$。所以，即使我们从不考虑$\varPsi$也不会影响最终的结果。另一个有趣的地方在于$\varSigma_w$只在$\varPsi_t$中出现，再看LQR定义中的$s_{t+1}=A_ts_t+B_ta_t+w_t$可以得出，即使在不知道噪音项协方差的前提下，也可以正确的求出最优策略（但不要忘了最优价值函数是受$\varPsi$影响的）。最后，这两点都是对这个特殊LQR模型（非线性动力系统）独有的，一旦我们对模型做出改变，则这两个特性将不复存在，也就是只有在这个例子中才有“最优策略不受噪音项协方差影响”。在后面讨论Kalman滤波的时候会用到这个性质。

## 8. 强化学习算法的调试

我们在[第十一讲](chapter11.ipynb)中，在关于机器学习算法的调试介绍过直升机模型提到：

1. 搭建一个遥控直升机模拟器，主要是对状态转换概率$P_{sa}$进行建模（比如通过学习物理知识，依照空气动力学原理编写模拟器；或者也可以收集大量试验数据，通过拟合这些数据一个线性或非线性模型，以得到一个用当前状态、当前操作表示下一步状态的函数）；
2. 选择奖励函数，比如$R(s)=\left\lVert s-s_\mathrm{desired}\right\rVert^2$（$s$代表直升机当前的位置，这里示例的函数表示直升机实际位置与期望位置的平方误差，我们在[上一讲](chapter18.ipynb)的LQR中已经使用过这个奖励函数了）；
3. 在模拟器中运行强化学习算法控制直升机并尝试最大化奖励函数，$\mathrm E\left[R(s_0)+R(s_1)+\cdots+R(s_T)\right]$，进而求得对应的策略$\pi_\mathrm{RL}$。

现在，假设我们已经按照上述步骤得到了控制模型，但是发现该模型比人类直接操作差很多，接下来应该怎么办呢？我们可能会：
* 改进模拟器（也许模拟器所用的模型不是线性或非线性的；也许需要收集更多数据来拟合模型；也许需要调整拟合模型时所使用的特征值）；
* 修改奖励函数$R$（也许它不只是一个二次函数）；
* 改进学习算法（也许RL并不适合这个问题；也许需要对状态进行更精细的描述；也许在价值函数近似过程中使用的特征值需要调整）；

对于调整算法这样的任务，我们有太多的选择方向，如果选择错误，则将会浪费大量的时间精力。

接下来我们假设：
1. 直升机模拟器足够精确；
2. RL算法能够在模拟器中正确的控制飞机，因此算法正确的最大化了预期总收益$V_t^{\pi_\mathrm{RL}}(s_0)=\mathrm E\left[R(s_0)+R(s_1)+\cdots+R(s_T)\mid\pi_\mathrm{RL},s_t=s_0\right]$；
3. 最大化预期总收益与模型正确的操作飞机强相关。

如果上述三条假设均正确，则可以推出$\pi_\mathrm{RL}$能够控制好真正的直升机。

于是，我们可以**依次**做出如下诊断：
1. 首先，如果$\pi_\mathrm{RL}$在模拟器中飞的很好，但是在实际飞机上却表现不好，则说明是模拟器的问题；
2. 然后，用$\pi_\mathrm{human}$表示人类操纵者在操纵直升机时使用的策略，比较人类和算法的策略所对应的价值函数，如果$V^{\pi_\mathrm{RL}}\lt V^{\pi_\mathrm{human}}$，则说明是RL算法的问题。（算法并没有成功的最大化预期总收益。）
3. 最后，如果经比较发现$V^{\pi_\mathrm{RL}}\gt V^{\pi_\mathrm{human}}$，则说明问题出在奖励函数上。（最大化这个奖励函数并没有使得飞行水平提高。）

以上仅是一个强化学习调试的例子，因为我们恰好找到了一个极优秀的直升机操纵者，如果没有的话则需要想出别的调试方法。在通常的问题中，我们都需要自己找出针对问题的有效的调试方法。

## 9. 微分动态规划（DDP: Differential Dynamic Programming）

继续使用遥控直升机的例子，假设我们已经有模拟器并可以通过模拟器知道$s_{t+1}=f(s_t,a_t)$（即下一个状态是一个关于当前状态及动作的函数），而且这个模拟器是非线性、确定性的，噪音项也比较小。我们现在想要让直升机飞出一些特定轨迹，于是：
1. 先写出这个轨迹：$(\bar s_0,\bar a_0),(\bar s_1,\bar a_1),\cdots,(\bar s_T,\bar a_T)$，这也称为**标准轨迹（nominal trajectory）**；（这个轨迹也可能来自一个很粗糙的控制算法，但是虽然粗糙，却也是描述的我们想要做出的动作，比如在空中做$90^\circ$转弯之类的动作。）
2. 然后，我们在这个轨迹附近线性化$f$得到：$\displaystyle s_{t+1}\approx f(\bar s_t,\bar a_t)+(\nabla_sf(\bar s_t,\bar a_t))^T(s_t-\bar s_t)+(\nabla_af(\bar s_t,\bar a_t))^T(a_t-\bar a_t)$，也就是说能够得到一个线性函数$s_{t+1}=A_ts_t+B_ta_t$。我们这是在课程中第一次使用LQR来处理有限时域上的非平稳的动态问题，尤其要注意的是这里的$A_t,B_t$是依赖于时间的（也就是这是一系列不相等的矩阵）；（即使标准轨迹来自一套很糟糕的控制，但我们仍然希望系统在$t$时刻的状态和动作能够近似于这个糟糕的轨迹。也许这个控制算法做的工作很马虎，但毕竟这个轨迹描述了我们想要得到的动作，也就是希望$(s_t,a_t)\approx(\bar s_t,\bar a_t)$。）
3. 有了线性模型，再使用LQR计算最优策略$\pi_t$，于是我们就会得到一个更好的策略；
4. 在模拟器中按照新的策略实现一个新的轨迹，也就是：
    * 用$\bar s_0$初始化模拟器；
    * 用刚学到的$\pi_t$控制每一步的动作$a_t=\pi_t(\bar s_t)$；
    * 最终得到一套新的轨迹$\bar s_{t+1}=f(\bar s_t,\bar a_t)$
5. 得到新的轨迹后，我们就可以继续重复上面的操作，不停的优化这个轨迹（也就是重复步骤2-5）。

在实践中能够知道，这是一个非常有效的算法，DDP实际上是一种局部搜索算法，在每次迭代中，找到一个稍好的点进行线性化，进而得到一个稍好的策略，重复这个动作最终就可以得到一个比较好的策略了。（“微分”指的就是每次迭代我们都会选择一个点并通过求导做线性化。）

## 10. 线性二次型高斯（LQG: Linear Quadratic Gaussian）

### 10.1 Kalman滤波（Kalman Filter）

现在介绍另一种类型的MDP，在这种MDP中，我们并不能直接观察到每一步的状态（在前面的MDP中我们都会假设系统的状态已知，于是可以计算出策略，它是一个关于状态的函数；在LQR中我们也是计算$L_ts_t$，状态都是已知的）。

我们暂时先不谈控制，来说说普通的不能观察到状态信息的动态系统。举个例子，我们对“使用雷达追踪飞行器”的过程进行极其简化的建模：线性模型$s_{t+1}=As_t+w_t$，其中$s_t=\begin{bmatrix}x_t\\\dot x_t\\y_t\\\dot y_t\end{bmatrix},\ A=\begin{bmatrix}1&1&0&0\\0&0.9&0&0\\0&0&1&1\\0&0&0&0.9\end{bmatrix}$（这是一个非常简化的模型，可以理解为一架在2D空间中移动的飞机）。

在这个简化模型中我们无法观察到每一步的状态，但假设我们可以观察到另一个量——$y_t=Cs_t+v_t,\ v_t\sim\mathcal N(0,\varSigma_v)$：比如$C=\begin{bmatrix}1&0&0&0\\0&0&1&0\end{bmatrix}$，则有$Cs_t=\begin{bmatrix}x_t\\y_t\end{bmatrix}$。这个量可能是雷达追踪到的飞行器的位置（只能得到位置相关的状态，而不能得到加速度相关的状态）。比如说在$\mathrm{xOy}$平面上，有一个雷达位于$x$轴某点，其视角沿$y$轴正方向大致对着飞行器，并依次观察到一系列飞行器移动过程中的不连续的点，因为雷达视角沿$y$轴正半轴方向，所以得到的点的坐标的纵坐标的噪音（方差）应该大于横坐标的方差，可以说这些点分别位于一个个小的高斯分布中，而这些高斯分布的协方差矩阵应该型为$\varSigma=\begin{bmatrix}a&0\\0&b\end{bmatrix},\ a\gt b$（即因为在$y$轴方向上的观测结果更容易出现偏差，所以等高线图中椭圆的长轴会沿着$y$轴方向。）而我们想要做的就是通过观测到的一系列坐标估计飞行器在每个观测点的状态$P(s_t\mid y_1,\cdots,y_t)$。

$s_0,\cdots,s_t;\ y_0,\cdots,y_t$组成了一个高斯分布的联合分布（多元正态分布），可以用$z=\begin{bmatrix}s_0\\\vdots\\s_t\\y_0\\\vdots\\y_t\end{bmatrix},\ z\sim\mathcal N(\mu,\varSigma)$表示，再结合[第十三讲](chapter13.ipynb)中了解的多元正态分布的边缘分布与条件分布的计算方法，我们就可以对$P(s_t\mid y_1,\cdots,y_t)$进行求解。虽然这个思路在概念上是可行的，但是在实际中跟踪一个飞行器会得到成千上万个位置，于是就会有一个巨大的协方差矩阵（期望向量和协方差矩阵都会随着时间、步数的增长呈线性增长，在计算过程中需要用到的边缘分布、条件分布都可能会使用协方差的逆，而计算协方差的逆的复杂度是矩阵规模的平方级别），所以对于计算来说可行性较低。

于是我们引入**Kalman滤波（Kalman Filter）**算法来解决这个效率问题。这个算法实际上是一个隐式马尔可夫模型，使用递推的方式计算，下面写出算法的步骤，算法的思路可以在[Hidden Markov Models](http://cs229.stanford.edu/section/cs229-hmm.pdf)中找到：
* 第一步称为**预测（predict）**步骤，使用$P(s_t\mid y_1,\cdots,y_t)$对$P(s_{t+1}\mid y_1,\cdots,y_t)$进行预测；
  * 有$s_t\mid y_0,\cdots,y_t\sim\mathcal N\left(s_{t\mid t},\varSigma_{t\mid t}\right)$；
  * 则有$s_{t+1}\mid y_0,\cdots,y_t\sim\mathcal N\left(s_{t+1\mid t},\varSigma_{t+1\mid t}\right)$；
  * 其中$\begin{cases}s_{t+1\mid t}&=As_{t\mid t}\\ \varSigma_{t+1\mid t}&=A\varSigma_{t\mid t}A^T+\varSigma_t\end{cases}$；
  * 需要说明的是，我们使用诸如$s_t,y_t$表示模型的真实状态（如为观测到的实际状态，已观测到的实际位置），而使用$s_{t\mid t},s_{t+1\mid t},\varSigma_{t\mid t}$表示由计算得到的值；
* 第二步称为**更新（update）**步骤，使用上一步预测的$P(s_{t+1}\mid y_1,\cdots,y_t)$更新$P(s_{t+1}\mid y_1,\cdots,y_{t+1})$，也就是有了预测结果之后再将对应的样本纳入模型：
  * 其中$s_{t+1\mid t+1}=s_{t+1\mid t}+K_{t+1}\cdot\left(y_{t+1}-Cs_{t+1\mid t}\right)$；
  * 而$K_{t+1}=\varSigma_{t+1\mid t}C^T\left(C\varSigma_{t+1\mid t}C^T+\varSigma_t\right)^{-1}$；
  * $\varSigma_{t+1\mid t+1}=\varSigma_{t+1\mid t}+\varSigma_{t+1\mid t}\cdot C^T\left(C\varSigma_{t+1\mid t}C^T+\varSigma_t\right)^{-1}C\varSigma_{t+1\mid t}$

  Kalman滤波算法的复杂度比计算协方差矩阵的逆低很多，因为采用了递推的计算过程，每当步骤增加，则只需多执行一步迭代，而且无需保留太多数据在内存中，相比于求一个巨大矩阵的逆而言，算法复杂度低了很多：
  $$
  \require{AMScd}
  \begin{CD}
  y_1 @. y_2 @. y_3 \\
  @VVV @VVV @VVV \\
  P\left(s_1\mid y_1\right) @>>> P\left(s_2\mid y_1,y_2\right) @>>> P\left(s_3\mid y_1,y_2,y_3\right) @>>> \cdots
  \end{CD}
  $$
  从这个草图可以看出来，当计算$P\left(s_3\mid y_1,y_2,y_3\right)$时，我们并不需要$y_1,y_2,P\left(s_1\mid y_1\right)$，每步计算只需要知道最新的观测值和上一步得出的概率估计即可。

### 10.2 LQG

将Kalman滤波与LQR结合起来就可以得到**线性二次高斯（LQG: Linear Quadratic Gaussian）**线性二次高速算法。在LQR问题中，我们将动作$a_t$加回到模型中：设非线性动力学系统为$s_{t+1}=As_t+Ba_t+w_t,\ w\sim\mathcal N(0,\varSigma_w)$，而且我们无法直接观测到状态，只能观测到$y_t=Cs_t+v_t,\ v\sim\mathcal N(0,\varSigma_v)$。现在使用Kalman滤波估计状态：
* 假设初始状态为$s_{0\mid0}=s_0,\varSigma_{0\mid0}=0$；（如果不知道确切初始状态，测可以大致估计初始状态$s_0\sim\mathcal N\left(s_{0\mid0},\varSigma_{0\mid0}\right)$，即使用初始状态的平均值和协方差的估计。）
* 预测步骤，$\begin{cases}s_{t+1\mid t}&=As_{t\mid t}+Ba_t\\\varSigma_{t+1\mid t}&=A\varSigma_{t\mid t}A^T+\varSigma_t\end{cases}$。如此，我们就“解决”为了状态观测不到的问题（这里的$A,B$都是已知项）；
* 控制步骤，按照LQR算法中的最优动作$a_t=L_ts_t$计算其中的$L_t$，并暂时忽略观测不到状态的问题；得到$L_t$后，选择$a_t=L_ts_{t\mid t}$作为最优动作（换句话说，因为不知道状态，所以我们对状态的最佳估计就是$s_{t\mid t}$）。

这个算法实际上是先设计了一个状态估计算法，然后再使用了一个控制算法，把两个算法结合起来就得到了LQG。
