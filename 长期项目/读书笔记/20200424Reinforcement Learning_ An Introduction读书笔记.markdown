# Reinforcement Learning : An introduction
这本书的中文版已经出了，我已经买了，在学校宿舍，没有带回来，非常可惜。
看看英文版也不错。

## 阅读记录

1. 20200424：记录第一次，已知第n次，目的是找到Q-learning更新的依据，尝试找到一种方法来对强化学习加速。


## Notation
大写字母表示随机变量，小写字母表示随机变量的值以及一些标量函数。
在bandit问题中：
* k 动作或摇臂的数量
* q*(a) 动作a的真实值
* Qt(a) 对q*(a)在t时刻的估计值
* Nt(a) 自时间t之后动作a被选到的次数
* Ht(a) learned preference for selecting action a

在MDP问题中：
* s,s' 表示状态
* a 表示动作
* r 表示奖励
* S 表示非终止状态的集合
* S+ 表示所有状态的集合，包括终止状态
* A(s) 表示状态s可执行的所有动作的集合
* R 所有可能奖励的集合
* t 离散时间步骤
* T,T(t) 一次循环的时间 final time step of an episode, or of the episode including time t
* At t时刻的动作
* St t时刻的动作 typically due, stochastically, to St-1 and At-1

等等，未完，有用到的再查

## 1 The Reinforcement Learning Problem

学习，最自然的概念就是与环境交互。在本书中我们会探索一种计算方式来通过与环境的交互进行学习。

### 1.1 Reinforcement Learning

强化学习问题包括学会做什么，即如何将状态映射到对应的动作从而最大化期望信号。这些是closed-loop问题，因为学习系统的结果会影响到它之后的输入。与机器学习相比，智能体可能并不知道它们能够采取什么样的动作以及会带来什么样的奖励，因此必须要去探索以及发现，从而获取最多的奖励。

强化学习问题的定义会在第三章用MDP来给出，但是基本的思路就是对于一个给定的智能体，用与环境交互的方式训练其实现某个目标，比如取得最大的长期收益。显然，这个智能体必须要能够感知这个环境，并在感知之后进行相应的行动，这正是MDP公式能够解决的问题。

强化学习并不是监督学习或非监督学习，强化学习所面临的一个重要问题是探索(exploration)和穷尽(exploitation)之间的平衡。为了获得最多的奖励，智能体会倾向于选择当前的最优解，但是这样子就不会探索环境，很可能会陷于局部的最优解中。问题在于这两者并不能共存，智能体必须要经过多次尝试才能够找到最优的选择。
另外一个重要的特性是强化学习会考虑the whole problem，关于一个面向目标的智能体与一个不确定的环境交互。而不是像很多机器学习问题那样先去考虑子问题，比如监督机器学习会只考虑哪些最终会有用的能力。
强化学习与心理学和神经科学高度相关，是所有形式的机器学习中最接近人类思维方式的一种。（所以后面会有专门的两章讲心理学和神经科学）

### 1.3 Elements of Reinforcement Learning
* policy：定义了智能体在给定时间下的行为模式。严格来说，policy是环境已知状态到动作的一个映射，在某些案例中，policy是一个简单的函数或查找表，其他形式可能是涉及到复杂计算的搜索过程。
* reward signal：定义了强化学习问题的目标。在每一个时间周期内，环境会给予强化学习智能体一个数字，这个数字被成为奖励reward。智能体唯一的目标就是最大化它所收到的总体奖励，因此奖励信号定义了好与坏的行为。其在生物学上，即为快乐或痛苦。因此，这个过程不能被智能体所改变，智能体只能够通过其动作改变产生的值，而不能改变这个系统(can not change the function that generate the signals)。
* value function：reward signal定义了短期的good，value function定义了长期的good。严格上来说，state的value是智能体所期望能够获得的所有的reward(total amount of reward an agent can expect to accumulate over the future, staring from that state)，因此values是一个长期的定义。从人的理解上来讲，reward是pleasure/pain，value是对自己未来有多pleasure或pain的长期判断。
    需要注意，我们只能够通过监控得到reward，从而估算出value，而动作选择是基于对value的判断的。这是更加困难的，reward是直接由环境给出的值，而values必须通过观测值反复修正和估计，这个过程将会持续整个智能体的生命周期。
* model of environment，即环境的模型，可以看作是对环境动作的模仿，或是对环境将会怎么做的估计。例如，给定一个状态和动作，这个模型可能会预测下一个状态与奖励，这是用于planning计划中的。使用model的模型被称为model-base方法，与之相对的就是model-free方法，在第8章会讨论会讨论model-base方法，只使用trial和error学习环境的模型并进行planning。

### 1.4 Limitations and Scope

相关：进化算法。如果policy的空间很小，而且策略很好找到，那么进化算法的效果会更好。

### 1.5 An Extended Example: Tic-Tac-Toe

井字棋。
在游玩的时候，我们会改变每个状态的values，动态地进行估计。一般来说，令s为当前状态，s'为之后的状态，V(s) = V(s) + alpha * (V(s')-V(s))，其中alpha是一个较小的正数，被称为step-size parameter，影响了学习率。这个更新规律也是temporal-difference learning method的一个例子，因为它是基于这个差V(s')-V(s)这两个不同时间的估计值的。
这样如果step-size parameter随着时间而减小的话，算法最终会收敛在一个最佳的策略，此时战胜对手的可能性是最大的。如果这个step-size parameter最终并没有收敛，结果也会很好。

这个例子旨在说明进化算法与基于value function的方法之间的区别。为了评估一个policy，进化方法会进行多次实验，与不同的对手进行交手或者对对手的行为进行建模，胜利的频率会给出这个策略的一个无偏的胜率估计。

井字棋的状态空间是小且固定的，然而强化学习也可以被用在大，甚至是无穷的空间上。比如Gerry Tesauro(1992,1995)将算法与人工神经网络结合，在10^20个状态中进行搜索。神经网络为程序提供了足够的泛化能力，这样在新的状态中选择动作时可以基于过去面临的相似状态的经验。强化学习系统在这种大的状态空间下运行的好坏取决于过去的经验的泛化结果。
在井字棋的例子中，我们没有任何关于该游戏的先验知识(prior knowledge)，但是强化学习绝不只是学习和智力的习惯，与之相反，先验知识可以在很多方式上参与并提高学习的效率。可以不需要模型，但是有模型也是可以的。
从另外的角度来说，强化学习可以完全不用模型，比如model-free系统可以考虑它们的环境是如何随着单个行为而发生变化。井字棋的例子就是model-free的，它没有对对手的行为进行建模。因为模型需要在比较准确的时候才能够有用，因此model-free的方案与很多复杂的方法相比有很多的优点。

### 1.6 Summary

### 1.7 History of Reinforcement Learning

不读了，如果对历史感兴趣可以了解一下。

### 1.8 Bibliographical Remarks

参考文献。
本书的方向，即强化学习最终的目标是最大化values(maximizing the expected value of the amount of reward an agent can accumulate over its future)，这个并不是唯一正确的方法，因为它忽略了量的变化，即risk。Risk-sensitive optimization同样是非常重要的领域，用于金融与最优化控制方向。相关理论包括Utility theory。

# Tabular Solution Methods

在这部分，用最简单的例子来讲解强化学习的基本概念，状态和动作空间都足够的小因此value function可以用数组来表示。在本例子中，方法总是能够找到准确地解决方案，即可以找到最优的函数值和策略。
这部分的第一部分讲解了只有单一状态下的解决方案，即bandit problem。第二部分讲解了MDP问题，以及Bellman方程和value function。接下来的三张描述了解决MDP问题的基础方法：动态编程、蒙特卡罗方法和时间差分方法。
* Dynamic Programming数学上很严谨，但是需要对环境有一个完全且准确的模型。
* 蒙特卡罗方法不需要准确模型，但是对于step-by-step incremental computation并不适用。
* temporal-difference方法不需要模型，且能够增量计算，但是对分析来说过于复杂。

剩下的两章讲述了如何将三种方法结合。

## 2 Multi-arm Bandits
多悬臂摇臂机：经典的nonassociative, evaluative feedback problem
* nonassociative： most prior work involving evaluative feedback has been done，从而避免了大部分强化学习问题的复杂性。
* evaluate feedback是与instructive feedback相对应的概念，后者用于监督学习，可以明确指出一个行为是否是好的

### 2.1 A k-Armed Bandit Problem

问题：你将重复面临一个有k种答案的选择，每一次选择会依据动作得到一个符合固定概率分布的以数值衡量的奖励。目标是最大化在给定时间范围内的奖励总数，比如1000步。
这就是k摇臂赌博问题，显然通过最大化选择有最高reward的摇臂即可获得最佳结果。
令t时刻动作At，相关的奖励为Rt，动作a的价值value为q*(a)
则有q*(a) = E(Rt | At = a)，即动作a的价值是在该时刻选择动作a的reward的期望。
显然，如果能够知道每个动作的value，这个问题就非常好解决了：只需要每次选择reward最大的摇臂即可获得最大值。但我们假设不知道所有的奖励，使得动作a的estimated value为Qt(a) 约等于q*(a)。
如果能够维护这个动作的价值的估计，这样就使得每一个时间点至少有一个动作的值是最大的，这就是greedy action。通过选择greedy action，就是exploiting现有动作的知识。如果选择nongreedy的动作，那就是exploring，因为这将会提升你对非贪心动作的认识。
explore存在的必要性是因为你不知道nongreedy中是否存在着比greedy action更好的结果，因为目前的值都只是估计值，会随着动作而进行改变。

### 2.2 Action-Value Methods

下面这条式子用一种比较自然的方式来估计动作的价值value，使用过去的选择。
![式2.1](pic/RLIntro_2_1.png)
其中1_predicate是一个随机变量，当predicate为true的时候为1，为false的时候为0。如果值为0，则设为某个默认值比如Q1(a)=0。该式实际上就是一个简单平均，被称为sample-average method。这只是一种简单的估计动作价值的方法，也不是最好的一种。
暂时放下如何估计的问题，来考虑如何根据这种估计方式来选择动作。
最简单的动作选择方式就是采用最大的价值估计值，即选择出一个动作A*t，使得Qt(A*t) = max_a Qt(a)，即使得Qt最大的动作。这被称为greedy action selection，如
![式2.2](pic/RLIntro_2_2.png)
这种方式可以深挖(exploit)当前的知识并最大化即时的奖励(reward)，几乎没有在其他动作上下功夫去探索它们之中是否有更好的情况。
一种简单的改变就是在大多数时候都是贪心的，但是在概率为epsilon的时候随机选择，这个被称为epsilon-greedy method。这样的好处是随着步数无限增加，几乎所有的动作都会被探索无数遍，从而使得Qt(a)收敛到q*(a)，这也暗示了选择最佳动作的概率会收敛到大于1-epsilon。
为了严格证明greedy和epsilon-greedy方法的有效性，我们使用一个测试问题来在数值上进行衡量。假设有2000个k=10的k摇臂问题，对于每个问题，如图2.1所示，动作的价值value服从均值为0、方差为1的高斯分布。
![图2.1](pic/RLIntro_Fig_2_1.png)

此时，当一个学习算法在t时刻选择动作At时，会从mean q*(At)、方差为1的正态分布中获得一个值来作为奖励Rt的值，如图2-1所示。这被称作10-armed testbed。
对于任何学习方法，我们可以使用其在该环境下1000步交互的表现和行为来进行衡量。

图2.2将greedy method与两种贪心算法(epsilon = 0.01/0.1)进行了比较。上述方法都是用sample-average technique对动作的价值进行估计，上面的表显示epsilon=0.1的时候在开始会比贪心算法稍微慢一点，但是后面显著优于。贪心算法每次只对最优进行深挖，所以会趋向于平均值。下图表示epsilon-greedy method更可能会取得最佳动作。
epsilon=0.01会比epsilon=0.1要慢一点，但最终结果会更好。因此也可能通过将epsilon随时间减少从而取得较好的结果。
epsilon-greedy的优势是依据于任务的，比如方差更大的时候，显然增加探索的可能性会取得更优的结果。其他一些假设也会改变结果，比如
![图2.2](pic/RLIntro_Fig_2_2.png)
### 2.3 Incremental Implementation
累增式的定义
Qn = (R1+R2+...+Rn-1)/(n-1)，显然可以通过维护一个全体reward的和来进行估计，但这样会带来CPU和内存上的压力。
![式2.3](pic/RLIntro_2_3.png)
以此得到bandit具体算法
![bandit](pic/RLIntro_bandit_algorithm.png)
式(2.3)的更新规则将会多次出现，其泛用形式如下：
![式2.4](pic/RLIntro_2_4.png)
后面的表达式（Target - OldEstimate）就是估计的误差。
需要注意，该方法中使用的StepSize参数是随着时间变化而变化的，在本文中我们使用alpha来代替这个参数，或alpha_t(a)，有时候会令它等于1/n。

### 2.4 Tracking a Nonstationary Problem

上述问题在一个静态环境下表现良好，但是如果摇臂会随着时间而改变，事情就会发生变化。在这种情况下，对更近的奖励进行加权是更好的选择，一种方式是使用一个常数的step-size parameter，比如下式
![式2.5](pic/RLIntro_2_5.png)
![式2.6](pic/RLIntro_2_6.png)
其中alpha是(0,1]的一个常数，我们把它称为加权平均weighted average，因为式(2.6)的两个权值之和为1.

### 2.5 Optimistic Initial Value

### 2.6 Upper-Confidence-Bound Action Selection

### 2.7 Gradient Bandit Algorithms

### 2.8 Associative Search (Contextual Bandits)

### 2.9 Summary

## 3 Finite Markov Decision

### 3.1 The Agent-Environment Interface

### 3.2 Goals and Rewards

### 3.3 Returns 

### 3.4 Unified Notation for Episodic and Continuing Tasks

### 3.5 The Markov Property

### 3.6 Markov Decision Processes

### 3.7 Value Functions

### 3.8 Optimal Value Functions

### 3.9 Optimality and Approximation

### 3.10 Summary

## 4 Dynamic Programming

### 4.1 Policy Evaluation

### 4.2 Policy Improvement

### 4.3 Policy Iteration

### 4.4 Value Iteration

### 4.5 Asynchronous Dynamic Programming

### 4.6 Generalized Policy Iteration

### 4.7 Efficient of Dynamic Programming

### 4.8 Summary

## 5 Monte Carlo Methods

### 5.1 Monte Carlo Prediction

### 5.2 Monte Carlo Estimation of Action Values

### 5.3 Monte Carlo Control

### 5.4 Monte Carlo Control with Exploring Starts

### 5.5 Off-policy Prediction via Importance Sampling

### 5.6 Incremental Implementation

### 5.7 Off-Policy Monte Carlo Control

### 5.8 Return-Specific Importance Sampling

### 5.9 Summary

## 6 Temporal-Difference Learning

### 6.1 TD Prediction

### 6.2 Advantages of TD Prediction Methods

### 6.3 Optimality of TD(0)

### 6.4 Sarsa : On-Policy TD Control

### 6.5 Q-Learning : Off-Policy TD Control

### 6.6 Expected Sarsa

### 6.7 Maximization Bias and Double Learning

### 6.8 Games, Afterstates, and Other Special Cases

### 6.9 Summary

## 7 Multi-step Boostrapping

### 7.1 n-step TD Prediction

### 7.2 n-step Sarsa

### 7.3 n-step Off-policy Learning by Importance Sampling

### 7.4 Off-policy Learning Without Importance Sampling: The n-step Tree Backup Algorithm

### 7.5 A Unifying Algorithm: n-step Q(sigma)

### 7.6 Summary

## 8 Planning and Learning with Tabular Methods

### 8.1 Models and Planning

### 8.2 Dyna: Intergrating Planning, Acting and Learning

### 8.3 When the Model Is Wrong

### 8.4 Prioritized Sweeping

### 8.5 Planning as Part of Action Selection

### 8.6 Heuristic Search

### 8.7 Monte Carlo Tree Search

### 8.8 Summary

# 2 Approximation SOlution Methods

## 9 On-policy Prediction with Approximation

### 9.1 Value-function Approximation

### 9.2 The Prediction Objective (MSVE)

### 9.3 Stochastic-gradient and Semi-gradient Methods

### 9.4 Linear Methods

### 9.5 Feature Construction

#### 9.5.1 Polynomials

#### 9.5.2 Fourier Basis

#### 9.5.3 Coarse Coding

#### 9.5.4 Tile Coding

#### 9.5.5 Radial Basis Functions

### 9.6 Nonlinear Function Approximation: Artificial Neural Networks

### 9.7 Least-Squares TD

### 9.8 Summary

## 10 On-policy Control

### 10.1 Episodic Semi-gradient Control

### 10.2 n-step Semi-gradient Sarsa

### 10.3 Average Reward: A New Problem Setting for Continuing Tasks

### 10.4 Deprecating and Discounted Setting

### 10.5 n-step Differential Semi-gradient Sarsa

### 10.6 Summary

## 11 Off-policy Methods with Approximation

### 11.1 Semi-gradient Methods

### 11.2 Baird's Counterexample

### 11.3 The Deadly Triad

## 12 Eligibility Traces

### 12.1 The lambda-return

### 12.2 TD(lambda)

### 12.3 An On-line Forward View

### 12.4 True Online TD(lambda)

### 12.5 Dutch Traces in Monte Carlo Learning

## 13 Policy Gradient Methods

### 13.1 Policy Approximation and its Advantages

### 13.2 The Policy Gradient Theorem

### 13.3 REINFORCE: Monte Carlo Policy Gradient

### 13.4 REINFORCE with Baseline

### 13.5 Actor-Critic Methods

### 13.6 Policy Gradient for Continuing Problems (Average Reward Rate)

### 13.7 Policy Parameterization for Continuous Actions

# 3 Looking Deeper

## 14 Psychology

### 14.1 Terminology

### 14.2 Prediction and Control

### 14.3 Classical Conditioning

#### 14.3.1 The Rescorla-Wagner Model

#### 14.3.2 The TD Model

#### 14.3.3 TD Model Simulations

### 14.4 Instrumental Conditioning

### 14.5 Delayed Reinforcement

### 14.6 Cognitive Maps

### 14.7 Habitual and Goal-Directed Behaviour

### 14.8 Summary

## 15 Neuroscience

### 15.1 Neuroscience Basics

### 15.2 Reward Signals, Reinforcement Signals, Values, and Prediction Errors

### 15.3 The Reward Prediction Error Hypothesis

### 15.4 Dopamine

### 15.5 Experimental Support for the Reward Prediction Error Hypothesis

### 15.6 TD Error/ Dopamine Correspondence

### 15.7 Neural Acotr-Critic

### 15.8 Actor and Critic Learning Rules

### 15.9 Hedonistic Neurons

### 15.10 Collective Reinforcement Learning

### 15.11 Model-Based Methods in the Brain

### 15.12 Addiction

### 15.13 Summary

### 15.14 Conclusion

### 15.15 Bibliographical and Historical Remarks

## 16 Applications and Case Studies

### 16.1 TD-Gammon

### 16.2 Samuel's Checkers Player

### 16.3 The Acrobot

### 16.4 Watson's Daily-Double Wagering

### 16.5 Optimizing Memory Control

### 16.6 Human-Level Video Game Play

### 16.7 Mastering the Game of Go

### 16.8 Personalized Web Services

### 16.9 Thermal Soaring

## 17 Frontiers

### 17.1 The Unified View
