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

* 在工程目录下，运行`./predict.sh`

该脚本包含数据提取，格式转换，数据分析，结果预测等部分运行的命令，所有的流程会有序进行，最终的结果会输出在工程目录下的`answer.txt`中。

### 依赖库

* #### pandas

    * 数据处理库，用于整理保存数据。

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
3. 注释详尽，文档齐全，使代码易于读懂。
4. 程序采用PEP8标准python代码规范编写，简洁明了。
5. 采用人工经验分析和机器学习理论相结合，大大加强了预测结果的可靠性。
6. 经验分析的部分朴素易懂，源自生活。

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 三、项目架构

```
    Src
    ├── predict.sh
    ├── Requirements
    │   ├── user_info.txt
    │   ├── product_info.txt
    │   └── behavior_info.txt
    └── src
        ├── Empirical_analysis
        │   ├── getdata.py
        │   └── E_predict.py
        ├── Xgboost_analysis
        │   └── X_predict.py
        └── tools
            └── get_behavior_info.csv
```

简单说明：

* Requirements：提供的数据集
* src：主要的实现代码
* predict.sh：包含整个项目运行的命令
* Empirical_analysis：经验分析
* Xgboost_analysis：机器学习模型分析
* tools：在比赛过程中自设的提取数据的实用脚本

生成文件说明：

* 最终预测出的结果`answer.txt`在工程根目录下。

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 四、API说明

* ### sklearn.model_selection.train_test_split
    * 作用：随机划分训练集和测试集
    * 一般形式：X_train, X_test, y_train, y_test = train_test_split(dataset, label)
    * 参数：dataset，所要划分的样本特征集；label，所要划分的样本结果
    * 返回值：划分好的结果

* ### xgb.DMatrix
    * 作用：加载数据
    * 一般形式：dmatrix = xgb.DMatrix(data)
    * 参数：libsvm格式的数据和二进制的缓存文件
    * 返回值：一个DMatrix类
    * 方法：
        * get_label()
            * 获得DMatrix的label
        * set_label(label)
            * 设定DMatrix的label
        * num_col()
            * 获得DMatrix的列数
        * num_row()
            * 获得DMatrix的行数

* ### xgb.train
    * 作用：训练模型
    * 一般形式：bst = xgb.train(plst, dtrain, num_round, evallist)
    * 参数：plst，booster参数；dtrain，要训练的数据；num_round，增强迭代次数；evallist，在培训期间要评估的项目列表
    * 返回值：一个booster类
    * 方法：
        * save_model(fname)
            * 保存模型
        * load_model(fname)
            * 读取模型
        * eval(data)
            * 评估模型，data为输入，返回值为评估结果
        * eval_set(evals)
            * 评估模型，evals为需要被评估的list，返回值为评估结果


<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 五、程序逻辑

### 主体流程

```
 +---------------------------------+
 | sed 's/\t/,/g' xx.txt > xx.csv  |
 +---------------------------------+
                 |                         +--------------+
                 |  getdata.py        |--->| E_predict.py |---|
                 v                    |    +--------------+   |
       +-------------------+          |                       |     +------------------+
       | user_behavior.csv | ---------|                       |---->| final_result.txt |
       +-------------------+          |                       |     +------------------+
                                      |    +--------------+   |
                                      |--->| X_predict.py |---|
                                           +--------------+
```

### 说明

1. 先处理txt文件，转换为好处理的csv文件。
2. 调用getdata.py，得到每个人的按照时间排好序的行为csv。
3. 调用E_predict.py和X_predit.py，用两种方法进行预测，输出csv格式文件。
4. 将结果转换为提交要求的txt格式。

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 六、Xgboost模型

我们首先处理数据文件，将用户的行为按照每个人分开，再按照时间排序，得到一个直观的`usr_behaviors.csv`文件，便于我们接下来的特征提取。

### 特征提取

我们的目标是预测未来三天用户的购买行为，经过简单的分析之后，发现有大量的行为都是浏览，这也与一般的用户在网购时的规律相同，这个数据的参考价值不是特别的大，因此我们把重点放在了收藏和加入购物车这两个行为上。

接下来我们定义两个概念：“用户对该商品的有效收藏和加购的次数”和“非邻近行为”。

用户对该商品的有效收藏和加购的次数：设该数为a，用户的购买该产品的次数为b， 则若用户最后一次的行为是购买，a = b，否则a = b + 1。

非邻近行为：指相邻两次行为的时间间隔大于30分钟。

最终从用户行为中，我们选取了以下的特征：

* 用户对该商品的有效收藏和加购的次数
* 用户收藏或加购后购买的概率
* 用户浏览次数
* 用户收藏次数
* 用户加购次数
* 用户非邻近浏览次数
* 用户非邻近收藏次数
* 用户非邻近加购次数
* 用户最后一次收藏或者加购后是否购买了该商品
* 用户收藏或加购的商品数
* 用户最后一次对该商品的操作时间距截止时间的天数。

这些特征基本围绕着购买的种种行为和事件概率。

接下来我们再从其他两个文件中，提取固有特征：

* 用户等级
* 宝宝年龄
* 宝宝性别
* 商品种类
* 商品价格

这些作为补充的特征值。其他的如用户生日性别等等经过观察发现大量用户没有填写，被我们丢弃，商品品牌等不是关注的重点，因此也没有加入特征。

### 模型构建

xgboost模型具有速度快、效果好、功能多等特点，同时树模型对特征处理的要求不高效果不错，不管是类别特征，连续特征效果都很好。而且在以往 Kaggle竞赛中，该模型屡次取得较好成绩。因此我们就使用了这个模型。

由于大赛提供了贝贝网31天的数据，预测后3天的购买行为。我们选取提供的数据中的最后3天的购买行为作为label，用前28天的行为进行训练，得出我们的训练结果，最后再用31天中的后28天作为预测集，来预测接下来3天的数据。

### 优化思路

我们在预测的时候，发现预测的结果中，且正负样本比例严重失衡，其中正样本（购买）特别少，为了提高训练速度，又不降低训练的准确度。我们对负样本进行了20次随机有放回的采样，分别训练20个分类器，并用20个分类器分别预测后投票得到最终结果。提高训练速度的同时也提高了准确度。

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 七、经验分析

### 模型说明

这次比赛没有完全依靠机器学习来构建模型，而在此之外增加了一个“经验分析”的部分。这是因为考虑到作为一个电商的购物网站，其用户的行为本身就具有很大的随机性，单纯地依靠选取整体的特征值来训练模型，其结果并不是十分理想。然而，对于单个用户来说，一个人自身的购物行为却是有一定规律可行的。再者，根据我们在日常生活中的切身体验来总结，我们同样可以寻找到很多规律。

例如，一个总是今天加入购物车，明天或是后天就买了的人，那么他在预测时间之前一两天加入购物车的商品，就极有可能在要预测的时间内购买。一个总是浏览要买的商品很多遍的人，那么他就极有可能在要预测的时间内购买他之前浏览了很多遍的商品。一个用户规律性地买某件商品，我们也可以看一看按照规律这三天是否会买。还有，高等级的用户相对低等级的用户而言，购买力就强大了很多，加入购物车次数多的人和少的人相比，买东西的概率也会高很多。

种种如此，我们就称这种单纯依照生活中的经验，或是对用户数据简单分析得到的经验来预测购买的方法为经验分析。最后的事实证明，经验分析法提供的数据可靠性非常高（相对于我们构建的Boosted tree模型来说），在我们最终提交的结果中起到了不可或缺的作用。

好在比赛提供的数据比较齐全，用户的行为数据和时间的信息都有，经过小队讨论，我们最终定下了三条路线。

第一种：重复购买商品

这里的重复，指连续购买同一商品超过一次的，我们计算出连续购买的时间间隔，依照最后一次时间加上这个时间是否在26-28号期间来判断购物行为。不过，这种思路做出来的数据比较少，不是主要的预测结果来源。

第二种：从其他行为到购买的时间段

这一种方案从行为上入手，我们计算出每个用户的：1.从第一次浏览到购买的时间区间。2.从第一次收藏到购买的时间区间。3.从第一次加入购物车到购买的时间区间。最后我们将前三类行为的时间都加上计算得到的到购买所需的时间区间，根据结果区间是否与26-28号重合来判断购物行为。

第三种：针对最后一次操作

这种情况针对最后一次操作离截止时间较近的用户，若用户最后一次加购或收藏的时间很接近截止时间，且用户此前买过商品或加购收藏的产品较少，我们就认为他会购买此商品。

最终，我们将这三类数据取并集，就得到经验分析法预测出的全部结果。

### 优化思路

优化当然也要动用我们在日常生活积累的经验。首先必须优化，必须去做区分的，就是用户的购买力。不同购买力的用户，其加入购物车购买的概率不能放在一起讨论，其加入购物车到会买的天数，也不尽相同。而购买力的直接反映就是用户等级，等级高的活跃用户，就是因为经常购买而得到的经验值。而我们的优化反映到数据上就是，对于不同等级的用户，设置不同的条件，同时不断调参来使F1趋于最大值。

调参的时候很要注重的一点就是，为了提高精确率，那么数据量不能一味增加，最终我们的数据量确定在3万条左右。

<br/><br/>

最终我们将两种模型得到的数据取并集，得到大约8万条数据。既有了机器学习模型提供的有效预测数据，同时又补充了训练得不到的经验数据集。

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 八、大赛感想

第一次参加数据挖掘的比赛，收获很多，熟悉了拿到数据以后进行数据清洗/特征提取/特征选择/模型构建/效果评估的整个过程，由于缺乏经验，期间走了很多弯路，也学到了很多东西。首先我们体会到了机器学习的强大之处，刚开始，我们考虑到用户的差异以及数据量的问题，我们选择了自己建立模型，以正常的购买习惯来进行预测，最后的精确率很高。但是召回率很低，并且我们很快发现用这种经验模型是有上限的，所以我们在后期又转回机器学习路线，使用xboost来训练，同时将我们的经验模型作为辅助，最后的结果还是相当好的。总的来说，这次的参赛经验还是相当宝贵的。

<br/><br/><br/><br/>

--------------------------------------------------------------------------------

## 九、贡献者

* [Yue Pan](https://github.com/zxc479773533)
* [LOOKCC](https://github.com/LOOKCC)
* [Q . J . Y](https://github.com/qjy981010)

From HUST, 想个队名真难

2017.10.15 Finished.
