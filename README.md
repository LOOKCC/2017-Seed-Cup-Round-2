# Shopping Prediction for beibei.com

<p align="center"> by Q . J . Y, LOOKCC, Yue Pan from HUST, 想个队名真难</p>

<br/><br/>

题目要求见： [doc](Requirements/Task.pdf)

--------------------------------------------------------------------------------

## 一、编译与运行

### 编译环境

* OS: Archlinux 4.13.5
* Language: Python 3.6.2

### 运行方法

* 在工程目录下，`cd src`进入源文件目录
* 在`src`目录里，运行`./install.sh`

该脚本包含数据提取，格式转换，数据分析，结果预测等部分运行的命令，所有的流程会有序进行，最终的结果会输出在工程目录下的`result.txt`中。

### 依赖库

* #### pandas

    * 数据读取，格式化的基本实现。

* #### numpy

    * 矩阵操作库，用于处理获得的表中的数据。

* #### scikit-learn

    * python的一个机器学习库，这里用作划分数据为测试集和训练集。

* #### xgboost

    * 大规模并行boosted tree的开源工具。Boosted tree为本次比赛中机器学习分析中用到的模型。

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 二、基本特点

1. 全部采用Python3编写，Python语言的轻便型和其大量的数学支持库和数据处理库使得其非常适合处理机器学习的问题。
2. 程序结构逻辑清晰，普适性强，可用于预测同类型的其他问题。
3. 注释详尽，文档齐全，所有的函数和类的内部均有文档字符串，涉及功能，参数，返回值，异常，使代码易于读懂。
4. 程序采用PEP8标准python代码规范编写，简洁明了。
5. 采用人工经验分析和机器学习理论相结合，大大加强了预测结果的可靠性。
6. 经验分析的部分朴素易懂，源自生活。

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 三、项目架构

```
    Src
    ├── result.txt
    ├── Report.pdf
    ├── Requirements
    │   ├── Task.pdf
    │   ├── user_info.txt
    │   ├── product_info.txt
    │   └── behavior_info.txt
    └── src
        ├── install.sh
        ├── user_info.csv
        ├── product_info.csv
        ├── behavior_info.csv
        ├── Empirical_analysis
        │   ├──
        │   └──
        ├── Xgboost_analysis
        │   ├──
        │   └──
        └── tools
            ├──
            └──
```

简单说明：
* result.txt：最终预测得到的结果
* Requirements：任务要求和提供的数据集
* src：主要的实现代码
* install.sh：包含整个项目运行的命令
* Empirical_analysis：经验分析
* Xgboost_analysis：机器学习模型分析
* tools：在比赛过程中自设的一些提取数据的实用脚本

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 四、API说明

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 五、程序逻辑

### 主体流程

```
 +---------------------------------+
 | sed 's/,/\t/g' xx.txt > xx.csv  |
 +---------------------------------+
                 |                         +------------+
                 |  getdata.py        |--->| predict.py |---|
                 v                    |    +------------+   |
       +-------------------+          |                     | sed +------------+
       | user_behavior.csv | ---------|                     |---->| result.txt |
       +-------------------+          |                     |     +------------+
                                      |    +------------+   |
                                      |--->|  train.py  |---|
                                           +------------+
```

### 说明

1. 先处理txt文件，转换为好处理的csv文件。
2. 调用getdata.py，得到每个人的按照时间排好序的行为csv。
3. 调用predict.py和train.py，用两种方法进行预测，输出csv格式文件。
4. 将结果转换为提交要求的txt格式。

### 核心算法介绍

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 六、Boosted tree模型

### 模型说明

### 优化思路

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 七、经验分析

### 模型说明

这次比赛没有完全依靠机器学习来构建模型，而在此之外增加了一个“经验分析”的部分。这是因为考虑到作为一个电商的购物网站，其用户的行为本身就具有很大的随机性，单纯地依靠选取整体的特征值来训练模型，其结果并不是十分理想。然而，对于单个用户来说，一个人自身的购物行为却是有一定规律可行的。再者，根据我们在日常生活中的切身体验来总结，我们同样可以寻找到很多规律。

例如，一个总是今天加入购物车，明天或是后天就买了的人，那么他在预测时间之前一两天加入购物车的商品，就极有可能在要预测的时间内购买。一个总是浏览要买的商品很多遍的人，那么他就极有可能在要预测的时间内购买他之前浏览了很多遍的商品。一个用户规律性地买某件商品，我们也可以看一看按照规律这三天是否会买。还有，高等级的用户相对低等级的用户而言，购买力就强大了很多，加入购物车次数多的人和少的人相比，买东西的概率也会高很多。

种种如此，我们就称这种单纯依照生活中的经验，或是对用户数据简单分析得到的经验来预测购买的方法为经验分析。最后的事实证明，经验分析法提供的数据可靠性非常高（相对于我们构建的Boosted tree模型来说），在我们最终提交的结果中起到了不可或缺的作用。

好在比赛提供的数据比较齐全，用户的行为数据和时间的信息都有，经过小队讨论，我们最终定下了三条路线。

第一种：规律性购买商品

这里的规律性，指连续购买同一商品超过一次的，我们计算出连续购买的平均时间，依照最后一次时间加上这个时间是否在26-28号期间来判断购物行为。不过，这种思路做出来的数据比较少，不是主要的预测结果来源。

第二种：从其他行为到购买的平均时间

这一种方案从行为上入手，我们计算出每个用户的：1.从第一次浏览到购买的平均时间。2.从第一次收藏到购买的平均时间。3.从第一次加入购物车到购买的平均时间。最后我们将前三类行为的时间都加上计算得到的到购买所需的平均时间，根据结果是否在26-28号来判断购物行为。

第三种：计算购买概率

后来的数据证明了这种思路是预测数据的主要来源。目标紧盯购买之前必须进行的重要行为--“加入购物车”。对每个用户，计算出他们对每样商品在加入购物车之后购买某件商品的概率P，然后针对25号，即预测时间之前的一天每个用户加入购物车的商品。再设置一个阈值δ，当P>δ时，就预测该用户会买这个商品。经过我们反复调参测试之后，兼顾准确率与召回率,我们认为这个δ设置在0.1~0.2比较合理。

最终，我们将这三类数据取并集，再筛掉已经发生的购买事件（这部分就是我们的模型不小心预测到的25号之前的购买行为），就得到经验分析法预测出的全部结果。

### 优化思路

优化当然也要动用我们在日常生活积累的经验。首先必须优化，必须去做区分的，就是用户的购买力。不同购买力的用户，其加入购物车购买的概率不能放在一起讨论，其加入购物车到会买的天数，也不尽相同。而购买力的直接反映就是用户等级，等级高的活跃用户，就是因为经常购买而得到的经验值。而我们的优化反映到数据上就是，对于不用等级的用户，设置不同的δ值，同时考察的天数也不同。比如高等级的用户可以考察多几天的加入购物车等。最后我们确定的参数就是购买概率阈值δ和对不同等级用户设置的考察购物车的时间。不断调参来使F1趋于最大值。

调参的时候很要注重的一点就是，要兼顾准确率和召回率，那么数据量不能一味增加，最终我们的数据量确定在8万条左右。

<br/><br/>

最终我们将两种模型得到的数据取并集，得到提交的结果。既有了机器学习模型提供的有效预测数据，同时又补充了训练得不到的经验数据集。

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 八、贡献者

* [Yue Pan](https://github.com/zxc479773533)
* [LOOKCC](https://github.com/LOOKCC)
* [Q . J . Y](https://github.com/qjy981010)

From HUST, 想个队名真难

2017.10.15 Finished.