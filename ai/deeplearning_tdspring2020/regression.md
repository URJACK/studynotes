# 台大课程1-Regression

以预测宝可梦的CP值为例子，记录下基础的回归原理。

讲解预测中的偏差值的来源原因，以及如何选取较好的模型、训练集的分配策略。

讲解常用的梯度下降算法、梯度下降的原理。

讲解为什么输入数据需要进行标准化。

以代码为例，实际验证之前的步骤。

## 预测宝可梦的战斗力

### STEP1模型

![image-20200831233855350](.\regression\image-20200831233855350.png)

![image-20200831234247210](.\regression\image-20200831234247210.png)

### STEP2损失函数

![image-20200831234950863](.\regression\image-20200831234950863.png)

图中，颜色越接近紫色，LossFunction的数值就越小

![image-20200831235300244](.\regression\image-20200831235300244.png)

### STEP3优化

![image-20200831235613972](.\regression\image-20200831235613972.png)

使用梯度下降

求某点斜率，总体目的是为了让loss变得更小

![image-20200831235924724](.\regression\image-20200831235924724.png)

参数w 更新的表达式，注意这里的**负号**：

**负号**的原因很明显，当求出的梯度为负的时候，说明更大的w可以拥有更小的损失。

反之，梯度求出为正的时候，说明更小的w可以拥有更小的损失

故，此处必须带有负号。

![image-20200901000044457](.\regression\image-20200901000044457.png)

对于两个参数，是类似的

![image-20200901000728234](.\regression\image-20200901000728234.png)

![image-20200901000923079](.\regression\image-20200901000923079.png)

对于线性模型而言，是不存在局部最优解的，求解的结果自然就是全局最优解。

具体求解过程：

![image-20200901001429596](.\regression\image-20200901001429596.png)

我们可以发现，不同的模型，最终能达到的效果也不一样

![image-20200901002115858](.\regression\image-20200901002115858.png)

更高次项的也不一定就能够取得特别突出的优势

![image-20200901002408193](.\regression\image-20200901002408193.png)

比如在4次项的时候，就已经出现了过拟合

![image-20200901002517405](.\regression\image-20200901002517405.png)

这里列出复杂模型与简单模型的损失对比，进而得出过拟合的结论

![image-20200901003936646](.\regression\image-20200901003936646.png)

实际上，这个模型对不同的宝可梦来说，并没有进行针对型的预测，所以所以我们需要重新设计模型

![image-20200901004555202](.\regression\image-20200901004555202.png)

### STEP4重设计模型

![image-20200901004657059](.\regression\image-20200901004657059.png)

当然，我们可以用一个式子来表示它

![image-20200901004914718](.\regression\image-20200901004914718.png)

重新设计模型后，损失比之前的更好

![image-20200901005206907](.\regression\image-20200901005206907.png)

### STEP5进一步优化

#### 更多参数进入模型

我们将宝可梦的所有参数全部代入，将它们全都融入模型之中去。

观察结果，虽然训练集上表现良好，但测试集上表现并不佳

![image-20200901005653949](.\regression\image-20200901005653949.png)

#### Regularization

我们在损失函数中，加入权重惩罚。这样就可以让函数更平滑，从而受输入的影响更小。

![image-20200901010026826](.\regression\image-20200901010026826.png)

以下，列出了常用λ对损失的影响：

所以我们需要适当的调整λ的数值

![image-20200901010410334](.\regression\image-20200901010410334.png)

## 基础概念

### Error的来源

bias和variance是error的来源

![image-20200901085907977](.\regression\image-20200901085907977.png)

![image-20200901090033277](.\regression\image-20200901090033277.png)

#### Variance

更高的Variance，距离期望值的偏差就更大。反之更接近期望值。

更高的bias会距离最优解的偏差值就越大。

![image-20200901090537420](.\regression\image-20200901090537420.png)

下面是一个variance的例子，模型越简单，受到样本数据的影响就越小

![image-20200901091030829](.\regression\image-20200901091030829.png)

#### Bias

![image-20200901091614149](.\regression\image-20200901091614149.png)

复杂的模型，往往能够正确的包括最优解，但可能具有较高的Variance。

简单模型往往具有较低的Variance，但是可能具有较高的Bias，即可能没有包括最优解。

### 优化方法

![image-20200901092445634](.\regression\image-20200901092445634.png)

优化Variance

![image-20200901093017886](.\regression\image-20200901093017886.png)

更多的数据往往能够获得更好的优化效果。

同样，Regularization平滑函数后，也往往能够取得不错的效果，但有可能会增加bias。

### 模型选取

![image-20200901093319594](.\regression\image-20200901093319594.png)

#### cross validation

尽管使用不同的模型，在测试集中可以取得不同的Error。即便在这里选用Error最小的模型，在实际应用真实的数据集中，也未必是最好的。

![image-20200901093726646](.\regression\image-20200901093726646.png)

可以先分别训练一部分数据，选出局部较小的loss的模型，然后选中该模型，重点进行训练。

训练完成后，再放入测试集进行计算loss。

#### N-fold cross validation

![image-20200901094117700](.\regression\image-20200901094117700.png)

## 梯度下降

最简单的梯度下降，来达到优化的方法如下：

![IMG_20200902_183720](.\regression\IMG_20200902_183720.jpg)

这里一定要注意，梯度是指的损失函数loss对各参数求的偏微分：

如果参数A方向的偏微分为正，那么如果对A + ΔA就会让损失loss变大。所以我们需要A - ΔA < A，让A左移，获得更小损失。

反之如果参数A方向的偏微分为负，那么A + ΔA就会让损失减小，所以我们需要A - ΔA > A，让A右移，从而获得更小损失。

各个方向的累计，就是梯度。这也是为什么，参数的移动方向与梯度方向是**相反的**，也就是梯度更新中**负号**的来源。

![IMG_20200902_184015](.\regression\IMG_20200902_184015.jpg)

### Adagrad梯度下降

该方法可以让学习率处于一个递减的过程，即，学习率刚开始比较大，而后面比较小。

σ这个参数的具体数值，详细见后文。

![IMG_20200902_184728](.\regression\IMG_20200902_184728.jpg)

σ的具体含义，见下图所示，他是一个梯度的累加。

![IMG_20200902_185012](.\regression\IMG_20200902_185012.jpg)

我们将学习率η与参数σ结合化简可以得出如下的参数更新公式

![IMG_20200902_190739](.\regression\IMG_20200902_190739.jpg)

你会发现这个更新的公式中，梯度既存在于分子，又存在于分母。

直观理解的原因是，这样在梯度突然变得特别大的时候，分子分母会相互抵消。

梯度在突然变得特别小的时候，变化的趋势也会趋近于0.

![IMG_20200902_191520](.\regression\IMG_20200902_191520.jpg)

### Stochastic梯度下降

传统的梯度下降中，我们对每批数据（数据个数为n），进行求损失，再计算梯度。

这里的梯度是对所有的样本进行计算的结果。

而Stochastic梯度下降中，我们对每批数据中的每一个数据，都单独求损失->计算梯度->并更新参数。

![IMG_20200902_192923](.\regression\IMG_20200902_192923.jpg)

### 泰勒展开-梯度下降的原理

![image-20200902220132097](.\regression\image-20200902220132097.png)

![image-20200902230149978](.\regression\image-20200902230149978.png)

梯度下降与泰勒展开的结合可知，当学习率较低的时候，也就有近似于泰勒展开的等式出现

![image-20200902230413482](.\regression\image-20200902230413482.png)

![image-20200902230757813](.\regression\image-20200902230757813.png)

### 单纯梯度下降的不足

1·saddle point

2·local minima

这里的两种情况均属于微分为0的情况

![image-20200902231310399](.\regression\image-20200902231310399.png)

## Feature Scaling

对于不同取值范围的输入，一定要注意去规范化这些输入数据

![IMG_20200902_193241](.\regression\IMG_20200902_193241.jpg)

一句话说明原因的话，取值范围整体偏大的输入，会更多的影响最终整体的梯度，这样会让另外的输入优化效果不明显。

使用一张图片可以更清晰的阐明优化不明显的原因：

在没有进行规范化之前，X2整体取值偏大，改变W2，对loss的影响会更明显。

而**梯度优化的方向**，几乎可以理解成**等高线的法线方向**，但可以得知，绝大多数情况下，法线方向并不能指向**最低的loss点**。

相反，进行规范化后，loss的等高线会更接近单位圆，此时的法线方向指向会更接近**最低的loss点**。

![IMG_20200902_193517](.\regression\IMG_20200902_193517.jpg)

具体的规范化方法可以使用该方法

每一项减去平均值后，再去除以标准差。

![IMG_20200902_193848](.\regression\IMG_20200902_193848.jpg)

## 模型训练

模型当中的测试集的处理过程，有以下几个注意点：

### 数据集的原始形式

#### header部分

train.csv中，第一行具有额外的标题（后文的测试集中，第一行是没有标题的）

读取代码即为：

```python
data = pd.read_csv(TRAIN_DATAPATH, encoding='big5')
```

而后文的读取代码（不包含标题）则为，需要指明header属性是不存在的：

```python
data = pd.read_csv(TEST_DATAPATH, header=None, encoding="big5")
```

![image-20200902232804244](.\regression\image-20200902232804244.png)

#### 无用属性的过滤

前三列的属性我们是不需要的，原始是27列，我们只需要后面的24列。

```python
data = data.iloc[:, 3:]
```

#### 无法识别的字段值处理

```python
data[data == 'NR'] = 0
```

#### 转为numpy类型

```python
raw_data = data.to_numpy()
```

### 数据集的最终形式

最终数据集整理后，输入x的形式是这样的N x (18 x 9) 维度的数据，其中9代表一共有九个小时，18代表一共是18类指标：

![image-20200902233908155](.\regression\image-20200902233908155.png)

显然，为了整理出数据集最后的这个形式，需要对原始的数据集进行一些关键的操作，其中一个比较重要的，是类似于卷积的概念。

原始数据是一个月只采集了20天，每天的数据是 18 x 24 进行排列的。

![image-20200902235347829](.\regression\image-20200902235347829.png)

经过这样的运算后，每一个月的数据就变成了下图所示的结构--一个18x480的矩阵。

该矩阵中，480列的数据是连续的480个小时，**十个小时为一组**，**前九个小时**可以看做输入，**后一个小时**就可以看做输出。

这样，480列就可以抽离出471组输入和输出。

这里的18行，分别代表不同的指标，于是**前9个小时的18个指标，均看做不同维度的输入**，而后一个小时的PM2.5，就看做输出。

于是**输入的维度为 9 x 18 == 162**。

![image-20200902235815790](.\regression\image-20200902235815790.png)

### 训练

在训练中，y' = x * w + b

但是我们为了方便简写为 y' = x * w

可以让x 的维度扩充一维x_d，并且扩充的这个维度x_d里的数值全是1。同样的，w也需要扩充一维w_d

这样x_d * w_d时，相当于就等于 w_d，它就相当于是bias了。

```
x = np.concatenate((x, np.ones((N, 1))), axis=1).astype(float)
```

### adagrad 优化方法

这个优化方法会依赖于一个叫做**adagrad**的变量，它是用来实现adagrad算法的关键所在（让学习率开始高，后面低）

详细的实现，参见**data_train**方法。

保存模型并还原模型之后，因为出现了loss的异常，所以被迫将adagrad的数值（优化器的参数，而不是模型的参数）也进行了保存。

### 代码

```python
import pandas as pd
import numpy as np
import math

TRAIN_DATAPATH = './data/train.csv'
MODELPATH = './model/weight.npz'


def data_process_x_y(month_data):
    # 9个小时的数据(包括PM2.5)作为输入，第10个小时的PM2.5为输出
    # 类似于卷积的操作，对480个小时的数据均进行这样的操作，最终可凑够471组数据
    x = np.empty([12 * 471, 18 * 9], dtype=float)
    y = np.empty([12 * 471, 1], dtype=float)
    for month in range(12):
        for day in range(20):
            for hour in range(24):
                if day == 19 and hour > 14:
                    break
                beginIndex = day * 24 + hour
                x[month * 471 + beginIndex, :] = month_data[month][:, beginIndex:beginIndex + 9].reshape(1, -1)
                y[month * 471 + beginIndex, 0] = month_data[month][9, beginIndex + 9]
    return x, y


def data_process_monthdata(raw_data):
    month_data = {}
    DAY_DIM = 24
    DAY_SPAN = 18
    MONTH_DAYS = 20
    MONTH_SPAN = DAY_SPAN * MONTH_DAYS  # 360
    for month in range(12):
        # 一个月的数据
        # 18 x 24 为一天的数据，这里是 18 x 480 ，一个月一共是20天的数据
        sample = np.empty([DAY_SPAN, MONTH_DAYS * DAY_DIM])
        for day in range(MONTH_DAYS):
            beginIndex = month * MONTH_SPAN + day * DAY_SPAN
            sample[:, day * DAY_DIM: (day + 1) * DAY_DIM] = raw_data[beginIndex:beginIndex + DAY_SPAN, :]
        month_data[month] = sample
    return month_data


def data_process_normalize(x: np.ndarray):
    mean_x = np.mean(x, axis=0)
    std_x = np.std(x, axis=0)
    print(mean_x.shape)
    print(std_x.shape)
    for i in range(len(x)):
        for j in range(len(x[0])):
            if std_x[j] != 0:
                x[i][j] = (x[i][j] - mean_x[j]) / std_x[j]
    return x


def data_process_split(data: np.ndarray):
    train_data = data[: math.floor(len(data) * 0.8), :]
    validation_data = data[math.floor(len(data) * 0.8):, :]
    return train_data, validation_data


def data_train(x, y, itertime: int, init=False):
    w: np.ndarray
    # 163 ( 162 + 1 ) 这里是为了提供额外的bias项目
    dim = 18 * 9 + 1
    N = x.shape[0]
    if N != y.shape[0]:
        print("sample numbers of x,y are not equal")
        return
    if init:
        w = np.zeros((dim, 1))
        adagrad = np.zeros([dim, 1])
    else:
        model = np.load(MODELPATH)
        w = model['w']
        adagrad = model['adagrad']
    x = np.concatenate((x, np.ones((N, 1))), axis=1).astype(float)
    learning_rate = 100
    iter_time = itertime
    eps = 0.0000000001
    for t in range(iter_time):
        # 这里权重矩阵，因为前期的预处理，已经包括了bias
        loss = np.sqrt(np.sum(np.power(np.dot(x, w) - y, 2)) / 471 / 12)
        if (t % 100) == 0:
            print(str(t) + "times, loss is :" + str(loss))
        gradient = 2 * np.dot(x.transpose(), np.dot(x, w) - y)
        adagrad += gradient ** 2
        w = w - learning_rate * gradient / np.sort(adagrad + eps)
    np.savez(MODELPATH, w=w, adagrad=adagrad)


def data_process_traindata(data):
    data = data.iloc[:, 3:]
    data[data == 'NR'] = 0
    raw_data = data.to_numpy()
    print(raw_data.shape)
    print(raw_data[0])
    return raw_data


def main():
    data = pd.read_csv(TRAIN_DATAPATH, encoding='big5')
    raw_data = data_process_traindata(data)
    # 一共十二个月的数据，类型是dict
    # 每个月的数据的形状均为18 x 480   -> 480 代表 480个小时（只记录了20天，而不是30天）
    month_data = data_process_monthdata(raw_data)
    # 转化为162维度的输入 （18个类型，9个小时：相当于不同小时的同类型的数据，也看成是不同的输入特征）
    x, y = data_process_x_y(month_data)
    # 对数据进行预处理
    x = data_process_normalize(x)
    # 将数据分成训练集与验证集
    train_x, validation_x = data_process_split(x)
    train_y, validation_y = data_process_split(y)
    # data_train(train_x, train_y, 1000, True)
    data_train(train_x, train_y, 10000)
    data_train(train_x, train_y, 10000)


if __name__ == '__main__':
    main()
```

## 模型测试代码

### 数据集的原始形式

显然，测试集中，数据的处理容易多了，只需要将18行的9列数据，合并到同一行来即可。

当然也别忘了x最后需要扩充一个**全为1的维度**，这样可以在表达式中省去bias项。

![image-20200903000841670](.\regression\image-20200903000841670.png)

### 代码

```python
import pandas as pd
import numpy as np
import math
from env_tf_2_3.pytorch_learn.regression import model

TEST_DATAPATH = './data/test.csv'


def data_process_testdata(data):
    data = pd.read_csv(TEST_DATAPATH, header=None, encoding='big5')
    data = data.iloc[:, 2:]
    data[data == 'NR'] = 0
    raw_data = data.to_numpy()
    return raw_data


def data_process_x(data: np.ndarray) -> np.ndarray:
    N = data.shape[0]
    DATA_SPAN = 18
    DATA_SINGLE_DIM = 9
    beginIndex = 0
    sample_num = N // DATA_SPAN  # 样本个数
    # 偏移量项，需要额外补足一个维度
    x = np.empty([sample_num, DATA_SPAN * DATA_SINGLE_DIM])
    for cursor in range(sample_num):
        for i in range(DATA_SPAN):
            x[cursor, i * DATA_SINGLE_DIM: (i + 1) * DATA_SINGLE_DIM] = data[beginIndex + i, :]
        beginIndex = beginIndex + 18
    x = np.concatenate([x, np.ones((sample_num, 1))], axis=1)
    return x


def main():
    data = pd.read_csv(TEST_DATAPATH, encoding="big5")
    raw_data = data_process_testdata(data)
    x = raw_data
    x = data_process_x(x)
    x = model.data_process_normalize(x)
    m = np.load(model.MODELPATH)
    w = m['w']
    y_pred = np.dot(x, w)
    print(y_pred)


if __name__ == '__main__':
    main()
```

