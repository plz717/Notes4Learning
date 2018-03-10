###Model-free 和 Model-based
Model-free——不尝试去理解环境, 不理解世界对于他的行为会怎么样反馈，环境给了它什么就是什么. 
Model-based——理解世界是怎么构成的，可以通过建立simulator来模拟现实世界对自己行为的反馈, 最后他不仅可以在现实世界中玩耍, 也能在模拟的世界中玩耍。

Model-free 的方法有很多：Q learning, Sarsa, Policy Gradients
Model-free 中, agent只能按部就班, 一步一步等待真实世界的反馈, 再根据反馈采取下一步行动. 而 model-based中， agent能通过想象来预判断接下来将要发生的所有情况. 然后选择这些想象情况中最好的那种. 并依据这种情况来采取下一步的策略, 这也就是 围棋场上 AlphaGo 能够超越人类的原因. 

###policy-based 和 value-based
policy-based：根据概率采取行动, 所以每种动作都有可能被选中, 只是可能性不同.
value-based：根据最高状态动作价值来选动作（更为铁定）；不适用于连续动作
将两者相结合：Actor-critic

###回合更新 和 单步更新
回合更新：一个episode结束后才针对所有动作更新。Monte-carlo learning 和基础版的 policy gradients 等 都是回合更新制,
单步更新：每个step都做更新，这样就能边玩边学习.（更有效率，更常用）Qlearning, Sarsa, 升级版的 policy gradients（如a2c,a3c,ddpg） 等都是单步更新制

分别适用于不同的场合：
-  单步更新
适用于那些在一个**episode中**，**时常**会得到**不同大小**的奖励或惩罚的情况，比如flappy bird，每穿过一个空隙，就得到一定的奖励。
-  回合更新
适用于那些一个**episode最后**才会得到奖励或惩罚的情况，比如下棋游戏，中间不管采取什么措施，只要最后没有结果输或赢，就难以判断这些动作是好还是坏。

考虑一个寻宝藏的问题（只有寻找到宝藏时才会得到奖励）：
-  如果采用单步更新：
虽然我们每一步都在更新, 但是直到获取宝藏时, 我们才为获取到宝藏的上一步更新为: 这一步很好,和获取宝藏是有关联的,而之前为了获取宝藏所走的所有步都被认为和获取宝藏没关系. 也就是说一个回合只对最终状态的前一个状态进行了更新。
-  如果采用回合更新：
虽然我要等到这回合结束, 才开始对本回合所经历的所有状态都添加更新, 但是这所有的步都是和最终取到宝藏有关系的, 所以一个回合会对所有步进行更新。这种情况下，回合更新会更效率一些.

###on-policy 和 off-policy
on-policy：必须本人在场, 并且一定是本人**边玩边学习**。如Sarsa和sarsa-lambda
off-policy：可以自己玩/看别人玩。可以从自己的经验/别人的经验中学习。不必边玩边学习，可以白天玩晚上学习。如Q-learning

###Q-learning
![Q-learning][Q-learning]
[Q-learning]: https://morvanzhou.github.io/static/results/ML-intro/q4.png
>新策略 = 旧策略+学习率x（最大目标值-估计值）

其中alpha为学习率，表示每次将误差中的多大比例加入到新策略中。
gamma衰减因子：
等于0时表示agent只看眼前利益， 不顾长远
等于1时表示很有远见，当前（s，a）下的价值就等于未来所有奖励的加和。
为了保证算法的收敛性，一般取0.99。（但是实际应用中发现取1也能使得算法收敛，尽管在数学上没有得到证明。）

之所以叫off-policy，是因为动作a'并没有真的被采取，只是想象了一下采取这个动作会使得下一状态的价值最大，以得到当前（s，a）最大的目标价值。

###Sarsa(0)
![Sarsa][Sarsa]
[Sarsa]: https://morvanzhou.github.io/static/results/ML-intro/s4.png
Sarsa在状态s'时跟Q-learning一样会根据Q函数计算出状态s'下，价值最大的动作a’.以此作为当前(s，a)的最大目标价值。**也就是说Q(s，a)的更新方式几乎是一样的！！**
为什么是几乎呢？因为Q-learning中的a'是完全由Q输出最大的那个动作决定；但Sarsa的a'还可能因为e-greedy策略以小概率选到随机动作。
最大的区别在于---->
Q-learning并不care计算最大目标价值时所选的a'
>s->s'

Sarsa真的采取了这个a'
> s->s',a->a'

为什么说Q-learning更冒险，Sarsa更保守呢？
因为Sarsa在s'时采取的动作是更新前计算出来的a'，更新前（s'，a'）的价值大不代表更新后也大呀～
Q-learning在s'时根据最新更新的策略计算最新的a'，也就是说它每个时刻都选择的是当前最新策略下最大价值的动作（以1-e概率）

###Sarsa-lambda
Sarsa(0):单步更新法, 在环境中每走一步更新一次，只更新当前步子的前一步.
Sarsa(1):回合更新法，一个回合结束对之前的所有步都更新一次，且更新力度相同。
Sarsa(0<lambda<1):一个回合结束，对所有步更新一次，且离最终结果越近的步子更新力度越大.相比单步更新更有效率。 


###Sarsa-lambda反向认识
![Sarsa-lambda][Sarsa-lambda]
[elijibility trace]: http://www.zhihu.com/equation?tex=E_0%28s%2C+a%29+%3D+0
![Sarsa-lambda][Sarsa-lambda]
[elijibility trace]: http://www.zhihu.com/equation?tex=E_t%28s%2C+a%29+%3D+γλE_%7Bt-1%7D%28s%2C+a%29+%2B+1%28S_t%3Ds%2C+A_t%3Da%29
体现的是一个结果与某一个状态行为对的因果关系，与得到结果最近的状态行为对，以及那些在此之前频繁发生的状态行为对对得到这个结果的影响最大。
![Sarsa-lambda][Sarsa-lambda]
[]: http://www.zhihu.com/equation?tex=E_t%28s%2C+a%29+%3D+γλE_%7Bt-1%7D%28s%2C+a%29+%2B+1%28S_t%3Ds%2C+A_t%3Da%29
https://morvanzhou.github.io/static/results/reinforcement-learning/3-3-1.png

初始时，对所有状态动作对的访问次数都是0， 因此全零初始化E矩阵

-  对每一个episode：
	-  对每一个t时刻的(S,A),到达下一状态(S',A')，那么经历的状态动作对就是（S,A）,一旦经历一次，就可计算出TD-error——>delta
	并使得E(S,A)增加一，代表当前episode中，对这个状态动作对(S,A)的访问次数增加了一
	既然我们经历完这个(S,A)后获得了奖励R，那么就要将此奖励归功于之前的所有步，如何分配奖励呢？
		-  对所有的状态动作对(s,a):
			-  如果s==S,a==A，那么说明这个状态动作对就是**获得奖励前一步的状态动作对**，其E(s，a)必然非负（至少为1），其Q(s，a)就得到一次较大更新。

			-  如果(s,a)！=(S,A)，即状态动作对并不是得到奖励前的状态动作对，它们也有可能对策略更新起到较大影响，为什么这么说呢？
				-  因为假设agent在之前遇到过一个状态动作对(S1,A1),那么其E(S1,A1)就非负了（等于1），假设在获得奖励R前，agent又**多次经历了与(S1,A1)完全相同的状态动作对**，那么Q(S1,A1)就会被多次更新，也就是说策略会受它更多影响。（类似神经网络中多次用相同样本和标签更新网络时，网络会对这个样本标签对拟合的更好）


