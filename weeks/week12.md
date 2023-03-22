# **Day 10 An Efficient and Robust Cloud-based DeepLearning with Knowledge Distillation**

## 论文摘要概述
这篇论文在云边知识蒸馏领域提出了一种新的神经元流形蒸馏（NMD）方法，其中学生模型模拟教师的输出分布，并学习教师模型的特征几何。此外，为了进一步提高基于云的学习系统的可靠性，我们提出了一种自信的预测机制来校准模型的预测，并在多个数据集上证明了其有效性。
## 论文框架模型
论文提出的框架模型如下所示:<br>
![image](https://user-images.githubusercontent.com/51207072/226775333-3b4a23c5-5a3d-4efb-bcd2-6e2abe80c484.png)<br>
浅蓝色的教师网络是一个标准的深度残差网络ResNet110，奶油色的学生网络是一个浅层网络ResNet20，
两个网络都有3个**残余块**（residual block），但基本块的数量不同。ResNet110在每个残差块中有18个基本块，但ResNet20只有3个。知识蒸馏发生在每个**残余块**之后。
## 蒸馏方法NMD
在假定相同批次处理数据的知识蒸馏训练中，有教师特征集ft∈Ft和学生特征集fs∈Fs，对于预定义特征提取函数ψ(.)可以提取为ψ(ft)和ψ(fs)（注意是将高维特征映射到一个保留关键特征特征的相同维度的低维空间），于是有每对的l2距离：![image](https://user-images.githubusercontent.com/51207072/226776901-037d526c-7ae6-4c79-95c7-41fa53cca95d.png)，于是根据以前的蒸馏方法以及新的这个特征函数有新的损失函数：<br>
![image](https://user-images.githubusercontent.com/51207072/226776992-c11ad181-cb8b-4810-b439-08c4cb986d57.png)<br>
α、β、γi（_个人见解这里的yi是对应特征提取函数的l2距离前的_）是控制每个分量权重的超参数。超参数λ的选择非常重要的。对于特征f，浅层特征则选择一个相对较大的λ值，深度低层特征选择一个较小的λ值。
### 线性特征流形
流形学习的观点是认为，我们所能观察到的数据实际上是由一个低维流形映射到高维空间上的。由于数据内部特征的限制，一些高维中的数据会产生维度上的冗余，实际上只需要比较低的维度就能唯一地表示。
也就是说，流形可以作为一种数据降维的方式；另外流形能够刻画数据的本质。也就是说。既然学习到了“将数据从高维空间降维到低维空间，还能不损失信息”的映射，那这个映射能够输入原始数据，输出数据更本质的特征。<br>
<br>
假定一个d维线性特征流形M ⊂ R^d 嵌入在 R^m中，可定义一个构造函数h：
![image](https://user-images.githubusercontent.com/51207072/226779658-cfad7a3e-97cf-43b3-bb73-8e01507b3492.png)<br>
给定N个从DNN学习到的特征fi，可能是采样来自d维线性特征流，并且带有噪音：
![image](https://user-images.githubusercontent.com/51207072/226780118-33cd467c-c730-4bfd-ad60-f8fce1dff20b.png)<br>
vi是第i个特征fi的特征流形，ϵi是误差。目标是根据fi估计未知的低维特征向量vi，即想用更低维的vi来表示学习特征。<br>
![image](https://user-images.githubusercontent.com/51207072/226789547-949ebe4f-d07a-4fa8-9971-8ca700e5f89c.png)
F = [f1，f2···，fN ]包括所有N特性，我们的目标是找到c，A和V最小化近似误差E V = [v1，v2，···，vN]，E=[ϵ1，ϵ2，···ϵN],e是所有的N维列向量。
关于这里论证特征流形的举例二维的例子这里不再介绍,这里假设了c = f ¯ = F e/N（F的平均值），并采用SVD分解的方法来求AV的表达式，结论如下：<br>
![image](https://user-images.githubusercontent.com/51207072/226790110-0e90594c-ea88-4732-bde3-02a09b33878c.png)<br>
![image](https://user-images.githubusercontent.com/51207072/226790132-466bd384-eabc-497a-bed6-fff9c6bacb12.png)<br>

### 深层特征的特征流形
深度学习中特征流形更加复杂，非线性化。非线性流形学习的关键困难是无组织的降维数据。论文认为对非线性深度特征的流形学习的目标是在不明确知道h（·）的情况下，从相应的特征fi或h（vi）重构vi。

