# 预备知识

## 基础

### 常用机器学习算法

![image-20200914232913271](.\preparing\image-20200914232913271.png)

实际上数据量较小的时候，不一定非得要采用ML的方式来解决，很多时候，只需人为观察一下，制定一些Rule说不定就能解决了。



![image-20200914233041492](.\preparing\image-20200914233041492.png)

### 常用机器学习工具

gensim：自然语言处理

pandas：数据的预处理

XGBoost：分类与回归

Natural Language Tookit：英文自然处理

Caffe：图像

![image-20200914233543790](.\preparing\image-20200914233543790.png)

### 解决问题的流程

关键：**数据（特征）**的重要性大于**模型**的重要性。

**了解场景和目标**

可以采集到什么样的数据，需要达成什么样的结果

**了解评估规则**

针对准确率，还是一些其他的标准？

**认识数据**

数据是否平衡？

**数据预处理**

将一些离群点调整权重。

**特征工程**

从什么样的特征来构建模型。分析可能对最终结果有影响的特征。

**模型调参**

**模型状态分析**

**模型融合**

### 数据预处理

![image-20200914235330579](D:\LearningNotes\ai\deeplearning_match\image-20200914235330579.png)

![image-20200914235547821](D:\LearningNotes\ai\deeplearning_match\image-20200914235547821.png)

对过大的数据进行scaling

对一些数据可以进行二分类：“贵”，“不贵”

对分类的数据进行onehot编码，缺省数据可以单独占一个类别。（当然不一定非得是onehot）

事先可以筛选“特征重要度”，如L1-based-selection，进而合适的分配权重。
