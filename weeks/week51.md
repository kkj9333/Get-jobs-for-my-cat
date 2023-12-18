#LLM & RL 随笔
## 随笔1
最近看了不少LLM和RL结合的论文，基本上都可以说是参考Guiding Pretraining in Reinforcement Learning with Large Language Models里的，然后将LLM生成的文本嵌入来参与到RL中。个人比较赞成的的分类方法是L-Model和L-Policy这种分法，比较直观的展示了LLM在决策上的应用方式。与MCTS结合的L-Model方法，不仅拥有惊人的准确率，而且有很高的可解释性，而另外一种基于多个LLM智能体构建的解释，描述，规划，选择方案，不仅可以给出目标让RL进行学习，而且还能给出RL的决策过程