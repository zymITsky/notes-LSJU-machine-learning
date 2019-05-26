
# 第十八讲：线性二次调节

首先回顾一下前面关于马尔可夫决策过程的内容。

* 我们用五元组$\left(S,A,\{P_{sa}\},\gamma,R\right)$定义MDP，我介绍了$0\leq\gamma\lt1$、$R:S\to\mathbb R$等等……
* 定义了价值函数，并介绍了使用值迭代算法求价值函数$\displaystyle V(s):=R(s)+\max_a\gamma\sum_{s'\in S}P_{sa}\left(s'\right)V\left(s'\right)$；（经过有限次的迭代会靠近最优解。）
* 在值迭代后$V$收敛于$V^*$附近，再通过计算$\displaystyle\pi^*=\arg\max_a\sum_{s'\in S}P_{sa}\left(s'\right)V^*\left(s'\right)$得到最优策略$\pi^*$。

在[上一讲](chapter17.ipynb)中，我们了解了对无限状态MDP的求解，本讲我们将介绍一些做了轻微改动的MDP的变型形式——首先是修改奖励函数$R$，然后是对折扣因子$\gamma$的调整。记得在上一讲中，我们提到了无限状态MDP无法直接求解（我们使用了对价值函数的近似来求解）；而在本将中，我们将介绍一种特殊的MDP——即使状态空间是连续的（无限状态）依然可以直接对价值函数进行求解。

## 5. 状态-动作奖励（State-action reward）

我们要介绍的第一个MDP的变型形式就是状态动作奖励，我们将奖励函数改为$R:S,A\to \mathbb R$，也就是从一对“状态-动作”到实数的映射。此时我们就得到了一个状态-动作序列：

$$
\require{AMScd}
\begin{CD}
s_0,a_0 @>>> s_1,a_1 @>>> s_2,a_2 \cdots 
\end{CD}\\
$$

根据这个序列，我们的总收益就应该是$R(s_0,a_0)+\gamma R(s_1,a_1)+\gamma^2R(s_2,a_2)+\cdots$，而我们的目标依然是找到一个能够最大化这个总收益的策略。可以看出这里的奖励函数变为一个关于状态和动作的函数（其实我们也可以减少一种变量，使其变回标准的MDP中的只关于状态的函数），使用状态-动作奖励可以使我们更直接的对“在不同动作中有不同成本”的问题进行建模（比如说对一个机器人来说，让机器人动起来的成本会高于让机器人静止的成本，此时给机器人一个静止在原地的指令就会对应更低的成本；或者比如对一个越野车来说，在泥泞道路上行驶比在柏油路上行驶成本更高，此时“在泥泞道路上行驶”的指令就更“贵”）。

这个变动对于MDP来说并不大，我们依然使用贝尔曼方程求解$V^*$：

$$
V^*(s)=\max_a\left(R(s,a)+\gamma\sum_{s'}P_{sa}\left(s'\right)V^*\left(s'\right)\right)
$$

在以前的价值函数中，$\displaystyle\max_a$的目标是未来收益，而现在因为“状态”奖励函数变为了“状态-动作”奖励函数，所以$\displaystyle\max_a$运算符移到了最外面。现在从状态$s$开始求最优价值函数时，最大化的目标分为两个部分：第一部分是新的即时奖励——在状态$s$下执行动作$a$的回报；第二部分是折扣因子$\gamma$乘以未来的预期总收益。这两个部分合起来就是$V^*$——最优价值函数——也就是取等号右侧的所有动作的最优值。

我们再来看值迭代，基本还是原来的算法$\displaystyle V(s):=\max_a\left(R(s,a)+\gamma\sum_{s'}P_{sa}\left(s'\right)V\left(s'\right)\right)$（仍然是用贝尔曼方程更新$V(s)$），迭代直到$V(s)$（近似）收敛于$V^*(s)$。

最后一步就是通过$V^*(s)$求最佳策略$\pi^*(s)$了，基本和以前一样$\displaystyle\pi^*(s)=\arg\max_a\left(R(s,a)+\gamma\sum_{s'}P_{sa}\left(s'\right)V^*\left(s'\right)\right)$

## 6. 有限时域马尔可夫决策过程（Finite horizon MDP）

我们要介绍的第二个MDP的变型形式就是有限时域MDP，这个MDP的五元组为$(S,A,\{P_{sa}\},T,R)$，其中$T$称为时域，它限定了MDP的最大步数，也就是在$T$步之后采取的动作将不会得到奖励。

$$
\require{AMScd}
\begin{CD}
s_0,a_0 @>>> s_1,a_1 @>>> \cdots @>>> s_T,a_T
\end{CD}\\
$$

那么在这种情形下，预期总收益就变成了$R(s_0,a_0)+R(s_1,a_1)+\cdots+R(s_T,a_T)$。与以往的目标相同，我们仍需要找到一个策略来最大化这个预期总收益，唯一的不同就是在$T$步之后MDP就结束了。这也说明策略有可能是**非平稳的（non-stationary）**（“平稳的”是指该项不依赖时间相关的变量），也就是说我们对最优动作的选择会根据所处时间的不同而不同。举个例子：
$
\begin{array}
{|c|c|c|c|c|}
\hline
+1&\cdot&s&\cdots&+10\\
\hline
\end{array}
$，假设机器人现在位于$s$的位置，距离左边的$+1$奖励比较近，距离右边的$+10$奖励比较远，而现在的MDP有个限制条件——不能超过$T$步。则这个例子中的机器人在做关于下一步动作的决策时会考虑：如果MDP刚开始，离结束$T$还比较远，那么它就可能决定向右走去取的$+10$的奖励；而如果已经进行到MDP末尾，剩下的步数并不一定能够到达$+10$奖励，那么它就可能决定向左走去取$+1$的奖励。

再来看非平稳状态转换概率$P_{sa}^{(t)}$，以前的MDP中有$s_{t+1}\sim P_{sa}$，即通过前一个状态的$s,a$就可以确定这个状态的分布，$P_{sa}$不会随时间变化（不依赖于时间相关的变量）。而在有限时域MDP中，不同的时间下状态转换概率是不同的，也就是说决定状态$s_{t+1}$的不只是$s,a$，同时也取决于它处在什么时间$t$。

再举几个例子：

* 在对于飞行器的建模中，飞行器会随着飞行消耗燃料，于是飞行器的总质量会随着时间的推移而变轻。于是对于飞行器的模型，我们对下一步所做的决定不只是取决于当前状态或动作，同时也取决于烧了多少燃料——即时间；
* 在飞机航线规划时，天气会随着时间的推移而不断变化，于是在好天气下某条航线的飞行成本与在恶劣天气下飞行成本是相差很大的，所以此模型同样收到时间因素的影响；
* 在汽车导航规划路线时，同一个区域在不同的时间车流量可能会有很大变化，因此某些区域的形式成本与时间有着直接关系。

最后，奖励函数也可能会随着时间改变，所以预期总收益会变为$R^{(0)}(s_0,a_0)+R^{(1)}(s_1,a_1)+\cdots+R^{(T)}(s_T,a_T)$。所以现在我们需要一个非平稳的随时间调整的策略。

最优价值函数$V_t^*(s)=\mathrm E\left[R^{(t)}(s_t,a_t)+\cdots+R^{(T)}(s_T,a_T)\mid\pi^*,s_t=s\right]$表示在状态$s$及时刻$t$下启动模型一直到结束步骤$T$期间的预期总收益，所以这个最优价值函数也取决于当前的时间以及所剩的时间。我们现在用贝尔曼方程写出最优价值函数的值迭代算法：

$$
\begin{align}
V_T^*(s)&=\max_aR^{(T)}(s,a)\tag{1}\\
V_t^*(s)&=\max_a\left(R^{(t)}(s,a)+\sum_{s'}P_{sa}^{(t)}\left(s'\right)V_{t+1}^*\left(s'\right)\right)\tag{2}
\end{align}
$$

$(2)$式表示，如果从时刻$t$状态$s$开始计算预期总收益，则应为“在$s,a$的即时奖励”加上“未来奖励”（当执行动作$a$后，MDP按照状态转换概率$P_{sa}^{(t)}$进入下一个状态$s'$，那么当MDP进入下一个状态$s'$时间自然应该累加，这时的预期总收益就应该是$V_{t+1}^*\left(s'\right)$，可以参考[第十六讲](chapter16.ipynb)由价值函数的递归定义得出贝尔曼方程的过程）的最大值，这个式子将$V_t$表示为$V_{t+1}$的递推式。而$(1)$式就是这个递推算法的启动项，它表示在时刻$T$只能得到该时间点的即时奖励，之后MDP立刻结束，没有未来奖励。计算的时候我们从$(1)$式开始，得到MDP的最后一项，然后使用$(2)$式向后递推依次得到$V_{T-1},V_{T-2},\cdots,V_0$。

最后再计算最优策略即可：

$$
\pi_t^*(s)=\arg\max_a\left(R^{(t)}(s,a)+\sum_{s'}P_{sa}\left(s'\right)V_{t+1}^*\left(s'\right)\right)
$$

最优策略和上面的最优价值函数一样，使用向后递推依次计算每个时间点对应的策略。

（再提一下折扣因子$\gamma$，在有限时域MDP模型中并没有出现这个参数，因为这个参数的作用从计算角度来说与参数$T$的作用非常相似。在标准MDP模型中，$\gamma$保证了MDP在一定步骤后的“未来”中，执行动作的奖励趋近于令；而在有限时域MDP模型中，$T$直接限定了“未来”步骤的总数。所以，我们通常在模型中只挑选这两个参数中的一个。）

接下来我们将结合状态-动作奖励MDP和有限时域MDP，再加上某些合理的假设，得到一个新的MDP模型，它能够优雅、高效的对大型MDP进行求解。

## 7. 线性二次调节（LQR: Linear Quadratic Regulation）

我们想要把状态-动作奖励MDP和有限时域MDP的思想（动态规划算法）应用到连续状态、甚至是连续动作空间中。仍使用五元组$(S,A,\{P_{sa}\},T,R)$定义MDP，注意这里我们使用有限时序中的$T$而不是折扣因子$\gamma$，其中：
* 状态为$S\in\mathbb R^n$；
* 动作为$A\in\mathbb R^d$；
* 将下一步作为关于当前状态及动作的线性函数$s_{t+1}=A_ts_t+B_ta_t+w_t$，此处的$w_t\sim\mathcal N(0,\varSigma_w)$是一个期望为零、协方差为$\varSigma_w$的高斯噪音项。$A_t\in\mathbb R^{n\times n}, B_t\in\mathbb R^{n\times d}$。需要留意的是我们在这里复用了符号$A$，这里的$A_t$是一个系数矩阵，在后面出现的$A$基本上都表示这个矩阵，而不是上面的动作集合；（在后面的介绍中，我们会假设$A,B,w$都是已知项，而我们的任务是给MDP找到一个最优策略。另外，在后面的证明中，我们会看到噪音项$w_t$并不十分重要。）
* $T$就是时域的限制；
* 奖励函数定义为：
    $$
    \begin{align}
    R^{(t)}(s_t,a_t)&=-\left(s_t^TU_ts_t+a^TV_ta_t\right)\\
    U_t&\in\mathbb R^{n\times n},U_t\geq0\\
    V_t&\in\mathbb R^{d\times d},V_t\geq0\\
    \therefore&\ s_t^TU_ts_t\geq0,v_t^TV_ts_t\geq0\\
    \implies&R^{(t)}(s_t,a_t)\leq0
    \end{align}
    $$
    在上面的定义中，$U_t,V_t$都是半正定矩阵，根据半正定矩阵的性质（可以参考[线性代数第二十八讲](http://nbviewer.jupyter.org/github/zlotus/notes-linear-algebra/blob/master/chapter28.ipynb)）就有奖励函数始终是一个非正数，它只会有成本而并没有奖励。举个例子，比如我们想让直升机模型中$s_t\approx0$，那么可以取$U_t=I,V_t=I$，则$R(s_t,a_t)=-s_t^Ts_t-a_t^Ta_t=-\lVert s_t\rVert^2-\lVert a_t\rVert^2$，这意味着如果直升机“远离原状态”（保持稳定）或“被来回猛拉”（防止高频率的反复操作），则奖励函数将以二次增长来惩罚模型，当然我们也可以修改矩阵$U,V$对角线上的元素以调整不同状态分量的权重。
    
（在本节有$A=A_1=A_2=\cdots,B=B_1=B_2=\cdots$，我们将假设MDP是静止的。）

### 7.1 非线性函数的线性近似

我们先来讨论线性动态规划的情形，$s_{t+1}=As_t+Ba_t$，其中一种建模方法我们在前面提过，就是做多次试验：

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
\end{CD}
$$

我们尝试不同的动作，记录测试得出的的各种结果，并使用线性回归拟合参数：

$$
\arg\min_{A,B}\sum_{i=1}^m\sum_{t=0}^{T-1}\left\lVert s_{t+1}^{(i)}-\left(As_t^{(i)}+Ba_t^{(i)}\right)\right\rVert^2
$$

也就是监督学习中的最小化“A,B”对数据的预测的误差。这是一种对线性动态系统的合理建模。我们再来介绍一种线性模型的建模方法——求非线性模型再将其线性化（linearize a non-linear model），仍然用倒置钟摆的例子：

假设有某个非线性模型$s_{t+1}=f(s_t,a_t)\begin{pmatrix}x_{t+1}\\\dot x_{t+1}\\\theta_{t+1}\\\dot\theta_{t+1}\end{pmatrix}=f\left(\begin{matrix}x_{t}\\\dot x_{t}\\\theta_{t}\\\dot\theta_{t}\end{matrix},a\right)$（$f$是一个非线性函数），也就是说，下一个时间点的状态将是一个关于某些当前状态和当前动作的函数，在本例中的动作就是一个实数，表示小车向左或向右的加速度的大小。 如果将这个式子看做$s_{t+1}$关于$s_t$的函数，那么我们可以做一个横轴为$s_t$，纵轴为$s_{t+1}$的图像，并绘出这个非线性函数。于是，当要对$s_{t+1}$进行预测时，就在横轴上找一个点$\bar s_t$，并在$\bar s_t$求函数的导数/梯度，于是就可以做出在点$\left(\bar s_t,\bar s_{t+1}\right)$处与非线性函数曲线相切的直线，而这条直线就可以近似为非线性函数在该点附近的线性表示。用数学符号表示：

* 假设$s_{t+1}=f(s_t)$（这里为了简化计算先不考虑动作$a_t$），于是有$s_{t+1}\approx f'\left(\bar s_t\right)\left(s_t-\bar s_t\right)+f\left(\bar s_t\right)$，这就把原来的非线性函数写成了一个关于$s_t$的线性函数，而$\bar s_t$是一个常数。最后，就是在式子里对应出矩阵$A,B$了。

现在我们再考虑另一个问题——这条直线能够在多大程度上近似原来的非线性函数？很明显，在$\bar s_t$点附近的近似都可以做到足够好，但是离的越远就差距就可能越大，就会变成一个糟糕的近似。所以，当我们使用LQR线性化非线性函数时，其中一个重要的参数就是用来确定线性化后函数在什么区间上有足够好的近似效果，放在倒置钟摆的例子中就是，我们希望这个系统的绝大部分时间都落在在这个“足够好的近似区间”内，这就是一次“合理的近似”。因此，从经验上讲，我们通常根据“系统大部分时间所在的区间”来确定在哪个状态附近对这个非线性函数做线性化处理。在倒置钟摆系统下，我们希望系统总是处于“零状态”，也就是大部分时间位于轨道的中央附近、小车的速度大约为零、同时杆能够的与轨道近似垂直——这就是一个我们希望做线性化的区间。

写出通用的线性化方程：

$$
s_{t+1}\approx f(\bar s_t,\bar a_t)+(\nabla_sf(\bar s_t,\bar a_t))^T(s_t-\bar s_t)+(\nabla_af(\bar s_t,\bar a_t))^T(a_t-\bar a_t)
$$

选择合适的$\bar s_t,\bar a_t$（在式子里，这两个量是常数）附近的区间做线性化，就可以将$s_{t+1}$近似的表示为关于$s_t,a_t$的线性函数。最后，我们可以将其化为$s_{t+1}=As_t+Ba_t$的形式。

### 7.2 LQR的求解

有了有线性近似非线性函数的思想，回到LQR的目标——在知道矩阵$A,B,U,V$组成的二次型奖励函数后后，找到能够最大化预期收益的策略。

同以前一样，我们要最大化奖励函数$R^{(0)}(s_0,a_0)+R^{(1)}(s_1,a_1)+\cdots+R^{(T)}(s_T,a_T)$，利用前面介绍过的“有限时域MDP”中动态规划算法的思路：
* 先计算出MDP的最后一步$V_T^*(s)$：
    $$
    \begin{align}
    V_T^*(s)&=\max_{a_T}R^{(T)}(s,a)\\
    &=\max_{a_T}\left(-s_T^TU_Ts_T-a_T^TV_Ta_T\right)\\
    &=-s_T^TU_Ts_T
    \end{align}
    $$
    因为我们定义$V_T$是半正定矩阵（$a_T^TV_Ta_T\geq0$），所以要保证上式取最大值，那么最好的方法就是不做任何动作（使$a_T^TV_Ta_T=0$）
* 对应的最后一步的策略为$\pi_T^*(s_T)=\arg\max_{a_T}R^{(T)}(s_t,a_t)=0$；

接下来进入动态规划的步骤，先来考虑如何从已知$V_{t+1}^*$向后递推出$V_{t}^*$：
* 在有限时域MDP的$(2)$式中，我们只需把求和换成积分（因为现在又是求连续状态的情形了）即可：
    $$
    \begin{align}
    V_t^*(s)&=\max_{a_t}\left(R^{(t)}(s_t,a_t)+\sum_{s_{t+1}}P_{s_ta_t}^{(t)}\left(s_{t+1}\right)V_{t+1}^*\left(s_{t+1}\right)\right)\\
    &=\max_{a_t}\left(R^{(t)}(s_t,a_t)+\mathrm E_{s_{t+1}\ \sim P_{s_ta_t}}\left[V_{t+1}^*(s_{t+1})\right]\right)
    \end{align}
    $$
    我们跳过了换成积分的步骤，直接将后面的项写为期望形式。实际上LQR具有性质：每一个价值函数$V_{t+1},V_{t},\cdots$都可以表为二次函数。
    
    假设$\displaystyle V_{t+1}^*(s_{t+1})=s_{t+1}^T\varPhi_{t+1}s_{t+1}+\varPsi_{t+1},\varPhi\in\mathbb R^{n\times n},\varPsi\in\mathbb R$，即$V_{t+1}^*$是一个关于$s_{t+1}$的二次函数，则选择适当的$\varPhi_t,\varPsi_t$就可以得到$\displaystyle V_t^*(s_t)=s_t^T\varPhi_ts_t+\varPsi_t$。
    
    那么在上面的$V_T^*(s)$中，$\varPhi=-U_T,\varPsi=0\to V_T^*(s_T)=s_T^T\varPhi_Ts_T+\varPsi_T$。
* 然后就是向后递推过程：
    $$
    V_t^*(s_t)=\max_{a_t}\left(\underbrace{-s_t^TU_ts_t-a_t^TV_ta_t}_{\textrm{(a)}}+\underbrace{\mathrm E_{s_{t+1}\ \sim\mathcal N(A_ts_t+B_ta_t,\varSigma_w)}\left[s_{t+1}^T\varPhi_{t+1}s_{t+1}+\varPsi_{t+1}\right]}_{\textrm{(b)}}\right)
    $$
    式中的$N(A_ts_t+B_ta_t,\varSigma_w)$就是$P_{s_ta_t}$，而期望中的$s_{t+1}^T\varPhi_{t+1}s_{t+1}+\varPsi_{t+1}$就是$V_{t+1}^*(s_{t+1})$。这个式子可以理解为找到动作$a_t$使$\textrm{(a)}+\textrm{(b)}$能够取到最大值，其中$\textrm{(a)}$表示处于状态$s_t$的即时奖励，而$\textrm{(b)}$表示在$s_ta_t$下$V^*$的期望，而$V^*$是下一步的最优价值函数。这个式子就是将奖励函数、状态转换概率、最优价值函数等带入后的最终结果。
    
    整个等号右边可以看做是对一个“大型”二次函数极大值的求解过程，我们和以前一样，求这个二次函数关于$a_t$的导数并将导数置为零，最终解出$a_t$。现在省略推导步骤，直接写出结果就有：
    $$
    a_t=\underbrace{\left(B_t^T\varPhi_{t+1}B_t-V_t\right)^{-1}B_t^T\varPhi_{t+1}A_t}_{L_t}\cdot s_t
    $$
    要留意的是，可以把前面的部分看做是一个“大型”矩阵$L_t$，于是，$a_t$就是$L_t$与$s_t$的乘积，即$a_t$是关于$s_t$的线性函数。
* 最后是求解最优策略，记得我摸在有限时域MDP中提到，$\pi^*$计算中$\arg\max$的部分就是$V^*$计算中$\max$的部分，而得到的也就是上面的$a_t$：

    $$
    \begin{align}
    \displaystyle \pi_t^*(s_t)&=\arg\max_{a_t}\left(-s_t^TU_ts_t-a_t^TV_ta_t+\mathrm E_{s_{t+1}\ \sim\mathcal N(A_ts_t+B_ta_t,\varSigma_w)}\left[s_{t+1}^T\varPhi_{t+1}s_{t+1}+\varPsi_{t+1}\right]\right)\\
    &=L_ts_t
    \end{align}
    $$
    也就是说，要得到最优策略，我摸只需要计算一个矩阵$L_t$，然后再乘以当前状态$s_t$即可。换句话说，最优策略是当前状态的线性函数。需要注意的是，这不是一个近似的函数，我们不会说“找到一个最优线性策略”、“找到最优策略然后用直线拟合该策略”，这不是用直线去近似最优策略，而是说最佳策略就是一条直线、最优动作就是当前状态的一个线性函数。
    
    而当我们求出$a_t$之后，把它代回$\displaystyle V_t^*(s_t)=s_t^T\varPhi_ts_t+\varPsi_t$，就会发现在这个二次函数中有：
    $$
    \begin{align}
    \varPhi_t&=A_t^T\left(\varPhi_{t+1}-\varPhi_{t+1}B_t\left(B_t^T\varPhi_{t+1}B_t-V_t\right)^{-1}B_t\varPhi_{t+1}\right)A_t-U_t\tag{3}\\
    \varPsi_t&=-\mathrm{tr}\varSigma_w\varPhi_{t+1}+\varPsi_{t+1}
    \end{align}
    $$
    即$\varPhi_t,\varPsi_t$是关于$\varPhi_{t+1},\varPsi_{t+1}$的线性函数，现在就可以计算$V_t^*(s_t)$了。其中$(3)$式也被称作**离散时间Riccati方程（discrete time Riccati equation）**。

对LQR做一个总结，算法如下：
* 初始化$\varPhi_t=-U_t,\varPsi_t=0$；
* 向后递推计算$\varPhi_t,\varPsi_t$，使用离散时间Riccati方程，将其当做关于$\varPhi_{t+1},\varPsi_{t+1}$的线性函数；（依次求出$t=T-1,T-2,\cdots,0$。）
* 计算$L_t$，这是一个关于$\varPhi_{t+1},\varPsi_{t+1}$的函数；
* 计算$\pi^*(s_t)$，这是一个关于$s_t$的线性函数，系数是$L_t$。