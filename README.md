## 简介
从 [UCI数据库](https://archive.ics.uci.edu/) 里获取了一份钓鱼网站数据集，该数据集包含11055个网站样本，每份样本记录了对应网站的31个特征，根据这些特征，设计了一个决策树二分类系统，用于判断给定样本是否为钓鱼网站。这也是本人的大学毕业设计，据此写了一份毕业论文，里面记录了这个项目的详细内容，现已上传至Paper文件夹下。同时，该项目已申请软著，登记号为：2023SR1597659，软件全称: 钓鱼网站决策树系统。

## 使用方式
```python
conda activate yourenv
pip install -r requirements.txt
cd ./DecisionTreeSystem
python main.py
```
注意：项目中使用了Graphiz，要使用pydot库，需先安装Graphviz，进入[官网下载](http://www.graphviz.org/download/)即可。
## 构建决策树
用不同数据构建决策树，整体的过程是相似的。先举个例子，假如有一筐水果，里面装了两种水果：梨和苹果。现在从筐中取出一个水果，识别其是否是苹果。我们选取水果的两个特征：一个是味道，味甜则取值为1，否则取值为-1；另一个是颜色，红色则取值为1，否则取值为-1。
接下来根据从这筐水果中收集到的数据构建决策树，根据决策的先后顺序，可以有两种决策树，如下：

![](https://gitee.com/Team317/pictures/raw/master/images/20220420091453.png)

这两种决策树的不同之处在于**构建决策树的顺序**，左图先判断颜色再判断味道，右图先判断味道再判断颜色。这两种方式的构建方式有什么区别呢？

决策树是基于已知的数据集构建的，每确定一个节点，数据集就会根据该节点的取值划分出多个子数据集，这些子数据集将用于构建以该节点为根节点的子树。

现在假定筐中苹果和梨的数量相同，90%的苹果为红色，90%的梨不为红色，且60%的苹果和梨都为甜味。

**如果选择颜色特征划分数据集，则根据颜色特征划分出的两个子数据集，同一子集中的样本基本属于同一类别，已经大致完成了分类，通过这两个子数据集构建的子树也更容易进行正确的分类，增加了子树分类的准确性；而如果先选择味道特征划分数据集，则划分后的两个子数据集内部样本类别依然很不统一，对子树正确分类的增益不大。**

通过这个例子，我想说明的是特征选择顺序对构建决策树的影响。应优先选择使数据集类别不确定性减少最大的特征，也即通过选定的特征划分得到的子数据集内部样本更倾向于属于同一类别。数据集类别的不确定性程度可以由信息熵进行衡量，这是我们接下来要讨论的内容。

### 信息熵

在信息论中是这样判断一个事件所包含的信息量的：**如果一个事件发生可能性越小，则它包含的信息量越大。极端的来说，如果一个事件100%发生，则该事件所包含的信息量近应为0**。事件`X`的发生有多种可能（例如明天的天气可能是天晴、多云、下雨等多种情况），以`P(Xi)`表示事件`X=Xi`发生的可能性，则事件`X=Xi`的信息量可以表示为：

$$
I(X_i) = log_2\frac{1}{P({X_i})}=-log_2(P({X_i}))\qquad\qquad(1)
$$

这称为一个事件的自信息，可以看到，当事件`P(X=Xi)=100%`时，即事件`X=Xi`一定发生时，其信息量为0。如果要衡量事件整体上的信息量，则可以使用`香农熵（Shannon entropy）`对整个概率分布中的不确定性总量进行量化。对于二值的事件`X`，若其发生的可能性为`p`，其香农熵计算方式为：

$$
H(X) = p*log_2(p) + (1-p)*log_2(1-p)\qquad\qquad(2)
$$

图形如下：

![](https://gitee.com/Team317/pictures/raw/master/images/熵.png)

可以看到，对于二值事件，当事件X发生的可能性为0.5时，该事件的熵达到最大，事件的不确定性也达到最大。更一般的，对于任意事件，其香农熵的定义方式为：

$$
Ent(x)=- \sum_{k=1}^{n} P(X_k)log_2P(X_k)\qquad\qquad(3)\\
$$

### 特征选择算法

回顾前面我们讨论的苹果和梨的例子，我们知道，如果决策的过程中系统的不确定能减少的更快，那么系统在后续做出正确决策的可能性也就更高。

**特征选择算法通过衡量不同特征对系统不确定性减少的程度，来选择构建决策树的特征。** 由于在实际情况中，事件的真实概率是未知的，只能通过数据计算出的频率来近似代替。

#### ID3算法

`ID3算法（Iterative Dichotomiser 3 迭代二叉树3代）`通过信息熵的变化大小来确定特征，先看它的计算方式：

$$
Gain(D,A_i) = Ent(D)-\sum_{i=1}^{k}\frac{|D_i|}{|D|}Ent(D_i)\qquad\qquad(4)
$$

`D`表示训练数据集，`Ai`表示的是数据集中的第`i`个特征，该特征有`k`种取值，当`A=Ai`时得到数据集`Di`；`Ent(D)`表示该分类问题的香农熵，由公式`(3)`进行计算；`Gain(D,Ai)`表示在特征`Ai`确定的条件下系统经验熵的变化量。在构建决策树时，将不同的特征`Ai`带入公式中，选择使`Gain(D,Ai)`最大的特征`A*`作为当前节点的选择的特征。

#### C4.5算法

`C4.5算法`是在`ID3算法`基础上进行的改进，考虑这样一种特殊情况：当前数据集有10个样本，特征`Au`有10种取值，特征`Au`的每种取值刚好有在数据集中有一个对应的样本。如果采用`ID3算法`，则当按特征`Au`对数据集进行划分时，由公式`(3)`计算得到每个子集的熵为0，从而`Gain(D,Ai)=Ent(D)`，这种情况下的信息增益达到最大。

所以`ID3算法`更容易选择取值结果个数多的特征，换句话说`ID3算法`并不能很好的反映出特征对系统不确定减少的影响程度。为避免上述情况，`C4.5算法`采用信息增益率进行特征选择，计算方式如下：

$$
Gain_ratio(D,A_i)=\frac{Gain(D,Ai)}{IV(A_i)}\qquad\qquad(5)\\\\
其中\quad IV(A_i) = -\sum_{i=1}^{k}\frac{|D_i|}{|D|}log_2\frac{|D_i|}{|D|}\qquad\qquad
$$

`IV(Ai)`和公式`(3)`是一致的，当特征的取值个数增加时，`IV(Ai)`也会增加。

#### CART算法

基尼指数的定义和信息熵的定义相似，数据集`D`的基尼指数定义如下：

$$
Gini(D) = -\sum_{i=1}^{k}\frac{|Di|}{|D|}(\frac{|Di|}{|D|}-1)=1-\sum_{i=1}^{k}(\frac{|D_i|}{|D|})^2 \qquad\qquad(6)
$$

**在`x=1`处，`f(x)=lnx`的一阶泰勒展开式为`x-1`，对比公式`(3)`，可以看出基尼指数和香农熵的定义是一致的。** 不同的是，基尼指数将指数运算换成乘法，使得计算更加高效。

特征`Ai`对系统基尼值的影响通过如下方式计算：

$$
Gini\_index(D,Ai)=\sum_{i=1}^{k}\frac{|D_i|}{|D|}Gini(Di)\qquad\qquad(7)
$$

这类似于公式`(4)`中的第二项，所以不难理解，在进行特征选择时，`CART算法`会选择使`Gini_index`最小的特征。


## 参考链接

[Phishing Websites](https://archive.ics.uci.edu/dataset/327/phishing+websites)：本项目的数据来源

[决策树相关开源项目](https://github.com/Erikfather/Decision_tree-python)：本项目参考了该项目的实现思路，并在此基础上进行修改和完善，有较大的改进。

[dot语法](https://en.wikipedia.org/wiki/DOT_%28graph_description_language%29#Attributes)：项目在使用pydot绘制决策树，dot语法可参考该文档。

