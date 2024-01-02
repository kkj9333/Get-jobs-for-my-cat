#LLM & RL 随笔
## 随笔1
最近看了不少LLM和RL结合的论文，基本上都可以说是参考Guiding Pretraining in Reinforcement Learning with Large Language Models里的，然后将LLM生成的文本嵌入来参与到RL中。个人比较赞成的的分类方法是L-Model和L-Policy这种分法，比较直观的展示了LLM在决策上的应用方式。与MCTS结合的L-Model方法，不仅拥有惊人的准确率，而且有很高的可解释性，而另外一种基于多个LLM智能体构建的解释，描述，规划，选择方案，不仅可以给出目标让RL进行学习，而且还能给出RL的决策过程.

## 一些MC可能有用的RL BaseLine
Baselines. We compare our approach with several baselines
in two categories: 1) end-to-end learning methods used in
prior work and in the official implementation provided, including BC [Kanervisto et al., 2020], SQIL [Reddy et al., 2019],
Rainbow [Hessel et al., 2018], DQfD [Hester et al., 2018],
PDDDQN [Schaul et al., 2015; Van Hasselt et al., 2016; Wang
et al., 2016]; all these baselines are trained within 8 million
samples with default hyper-parameters. 2) The top solutions
from MineRL 2019&2020&2021 including SEIHAI [Mao et
al., 2021] (1st place in MineRL 2020) and ForgER [Skrynnik
et al., 2021] (1st place in MineRL 2019).

## 一些有趣的想法

目前一些深度学习算法，采用自监督+少样本微调来进行学习，但是这样貌似只适合小样本问题。
在人类学习的过程中，聪明人总是很快学习到某种问题，并能够举一反三，这是聪明之处。前者其实是一个快速过拟合的能力，后者举一反三既要考虑泛化能力，也需要考虑快速能力。自监督是拥有一定程度上提高模型对数据集的泛化能力的，如果我们先过拟合一个数据集，然后用自监督使模型拥有举一反三的能力，是不是很好？这个在强化学习这里面，就是可以在已经学到某个任务的间隙，自监督来提供任务的泛化能力，这样就能像人类一样举一反三了？
这是不是一个很好的故事hhh
