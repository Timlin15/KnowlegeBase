进入深度 RL 后，model free 方法开始变得愈发重要，因为对真实世界建模绝非易事。

## 价值函数

当动作，状态空间无限扩大后，没有了一个表格式的 model 后，最重要的问题是如何表达原来查表可得到的价值函数 $q(s,a)$ 来指导动作选取。可以使用函数来状态动作价值函数的估计，在训练中采用近似参考值的方法来训练。
$$
J(w)=\mathbb{E}\lbrack(v_{\pi}(S)-\hat{v}(S,w))^{2}\rbrack,
$$
这个目标函数期望是按照随机变量 $S \in \mathcal{S}$ 来计算的，$S$ 的分布情况会影响期望的计算。有两种方法来规定 $S$ 的分布：

第一种是平均分布，平等地对待每一个状态，赋予 $\frac 1n$ 的权重：
$$
J(\imath w)=\frac{1}{\pi}\sum_{s\in S}(\nu_{\pi}(s)-\hat{\upsilon}(s,w))^{2}.
$$
但是这种方法有一种平均主义的问题：把所有状态都等权计算，这让很少访问到的状态同经常访问到的状态一样重要。

第二种是这章的重点，是应用稳态分布（*stationary distribution*）。如果你记录 agent 在足够多步里分别待在每个状态的比例，这个比例就近似于 stationary distribution $d_\pi(s)$，满足$\sum_{s} d_\pi(s) = 1$。形式上来说，策略 $\pi$ 作用于 MDP 后，状态的转移构成一条马尔可夫链，其转移矩阵为：
$$
P_\pi(s'|s) = \sum_a \pi(a|s) p(s'|s, a)
$$
如果一个分布 $d_\pi$  满足：
$$
d_\pi(s') = \sum_{s} d_\pi(s) P_\pi(s'|s)
$$
也就是说，用 $d_\pi$ 作为当前分布，经过一步转移后分布不变，那它就是 stationary distribution。
$$
J(w)=\sum_{s\in S}d_{\pi}(s)\bigl(v_{\pi}(s)-\dot{v}\bigl(s,w\bigr)\bigr)^{2},
$$

有了目标函数即可利用梯度下降来求函数参数的最优值：
$$
\begin{aligned}
w_{k+1}&=w_{k}-\alpha_{k}\nabla_{w}J(w_{k}),\\
&=\,\nabla_{w}\mathbb{E}[(v_{\pi}(S)-\hat{v}(S,w_{k}))^{2}]\\ 
&=\,\mathbb{E}[\nabla_{w}(v_{\pi}(S)-\hat{v}(S,w_{k}))(-\hat{v}(S,w_{k}))^{2}]\\ 
&=\,-\,2\mathbb{E}[(v_{\pi}(S)-\hat{v}(S,w_{k}))\nabla_{w}\hat{v}(S,w_{k})].\\
&=w_{k}+2\alpha_{k}\mathbb{E}[(v_{\pi}(S)-\hat{v}(S,w_{k}))\nabla_{w}\hat{v}(S,w_{k})],
\end{aligned}
$$

而这些公式中的 $v_{\pi}(s)$ 表示的是真实的奖励函数，我们无法获取这个值，只能通过