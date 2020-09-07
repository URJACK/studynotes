# 台大课程2-Classification

## 引入

### 线性回归

如果使用回归的方式来做分类：

我们对二维的参数，尝试拟合一条直线。处在这条直线上的点，函数值应该为0。

显然，任何一组二维输入[x1,x2]，都会存在大于0，小于0的情况，我们将大于0的归为一类，小于0的归为一类。



但用一条直线来进行分类，显然存在的一个问题是，如果有一些样本数的数值过大的话，这些过大的数值会将拟合的直线给“拉偏”，从而使得最终的结果达不到预期。

![image-20200903093123025](.\classification\image-20200903093123025.png)

### 替代方案

一个替代方案是，设置一个函数**f(x)**：

```
g(x) > 0归为class1
g(x) <= 0归为class2
```

使用该函数，我们对每一个样本进行预测，只要分类不同，就进行一次计数
$$
样本x^i\\
f(x^i)\neq y^i
$$
统计最终的个数。就是损失函数。

![image-20200903093420357](.\classification\image-20200903093420357.png)

## 盒子问题

### 引入

这里使用盒子问题作为分类方法的引子

事件A=“抓出的蓝球，来自B1这个盒子。”。

如果我们需要求的问题是，事件A的概率。

那么计算方式如下：

![image-20200903093604989](.\classification\image-20200903093604989.png)

### 类比分类问题

那么这个盒子的模型，可以类比到分类问题中去。

事件B=“输入的x，属于C1这个分类”

求事件B的概率。

那么计算方式如下：

![image-20200903093859093](.\classification\image-20200903093859093.png)

### 公式中的P(C1),P(C2)

显而易见，可以等价的列写出类似事件的概率公式，比如“输入的x，属于C2这个分类”的概率公式。

现在我们需要解决的是，解决公式中一些变量的来源

首先P(C1),P(C2)这两个数值是非常容易求得的，我们只需统计训练集中的对应分类的个数，计算他们的百分比，就可以求得这两个数值了。

![image-20200903094304390](.\classification\image-20200903094304390.png)

### 公式中的P(x|c1),P(x|c2)

还差P(x|c1),P(x|C2)这两个家伙无法求得。他们两个的含义分别是：

c1这个分类中，能取到x的概率是多少。

c2这个分类中，能取到x的概率是多少。



![image-20200903094940771](.\classification\image-20200903094940771.png)

#### 高斯分布

为了计算这两个条件概率，需要引入高斯分布。

高斯分布包含了μ和Σ两个参数。μ的维度与x的维度相等（假设为n），不难推出Σ的维度是(n , n)。
$$
f_{μ,Σ}(x) = ....
$$
我们将输入代入上述表达式，就可以得到该x，在当前分布下的一个概率值。也即P(x|c1)

当然，每一个不同的分类，就有一个不同的高斯分布。

![image-20200903095440780](.\classification\image-20200903095440780.png)

当然，为了优化每一种类别的高斯分布，以优化A类别为例，假设A类别的所有样本个数有79个，那么为了优化这个高斯分布，我们需要将Σ和μ参数取按如下公式取值。

![image-20200903095730459](.\classification\image-20200903095730459.png)

尝试计算这个结果，就可以得到Σ和μ参数。

![image-20200903095821967](.\classification\image-20200903095821967.png)

#### 求得条件概率

有了Σ和μ参数，条件概率就可以求得了

代入计算就可以求出P(c1|x)

![image-20200903100108157](.\classification\image-20200903100108157.png)

绘制出分布情况，我们发现的话两个特征其实对于整体预测准确率并不高。（显然，可能**防御力**和**SP防御力**这两个数值与宝可梦的**种族**这个属性并没有太多的关系）

即便放入7个特征（准确率依然只有54%）准确度依然不高。

![image-20200903100444032](.\classification\image-20200903100444032.png)

### 优化方案

#### 共用Σ

对之前的模型进行优化，我们将两个高斯分布**共用同一个Σ**，当然μ还是使用各自的μ。

共用的Σ取值公式如下图所示。

![image-20200903155840674](.\classification\image-20200903155840674.png)

我们发现共用了一个Σ后，图像的边界变为了一条直线（在后文经过数学推导后，会理解为什么是一条直线）

使用共用Σ后，7个维度的分类过程中，准确率从54%升到了73%，这说明共用Σ是有效的。

![image-20200903160313008](.\classification\image-20200903160313008.png)

这种各个输入维度的数据互相之间如果是毫无联系的，可以称作朴素贝叶斯分类。

![image-20200903161722343](.\classification\image-20200903161722343.png)

#### 公式推导

经过公式推导，我们可以得出
$$
P(C1|x) = \sigma(wx + b)
$$
以下是推导过程

![image-20200903174000029](.\classification\image-20200903174000029.png)

![image-20200903174240944](.\classification\image-20200903174240944.png)

![image-20200903174333222](.\classification\image-20200903174333222.png)

![image-20200903174528604](.\classification\image-20200903174528604.png)

多个高斯分布使用相同的Σ

![image-20200903174914466](.\classification\image-20200903174914466.png)

## 逻辑回归

由之前的推导，我们可以得知，一个二分类问题就可以等价于是一次逻辑回归。

![image-20200904095917561](.\classification\image-20200904095917561.png)

回顾之前的线性回归，逻辑回归与线性回归相比，仅仅在于输出处增加了一个sigmoid函数。

![image-20200904100132404](.\classification\image-20200904100132404.png)

### 损失函数的推导过程

#### 函数的好坏评价标准

如果我们能将所有C1类别的点都判别为1，所有C2类别的点都判别为0。

那么我们的模型就应该是比较最好的。



但显然实际的模型应当存在偏差值。所以函数的好坏定义，可以按如下方式进行衡量好坏：

类别为C1的点，因为本来应该被模型判别为1，所以我们就用它的概率值P(X1)作为好坏的标准。

类别为C2的点，因为本来应该被模型判别为0，所以我们用1 - P(X2)作为好坏的标准。

将每个样本点的标准相乘，我们就可以得到整体的好坏标准

![image-20200904100436163](.\classification\image-20200904100436163.png)

#### 损失函数的得出

当然，我们发现这个标准是越大越好，如果是损失函数的话，显然是越小越好。

所以我们在原有的L(w,b)之前加上了负号，同时，为了尽可能的使用加法运算，我们在前面加上了ln函数，这样就可以把原有的乘法运算转化为加法运算。

当然，为了能让**损失函数最终能用一个表达式**就可以表示，这里需要引入一个**前提条件**：

分类C1的标签值为1，分类C2的标签值就为0.（当然，他俩反过来也可以）

当引入了这个条件后，我们的公式就可以用一个单独表达式进行表示了。

![image-20200904101347762](.\classification\image-20200904101347762.png)

最终的损失函数如下所示

这个损失函数其实是一个交叉熵的特殊情况。

![image-20200904101753123](.\classification\image-20200904101753123.png)

#### 损失函数的拓展问题

这里引入了一个问题：

逻辑回归中，如果使用平方差作为损失函数可不可以呢？因为按照线性回归损失函数的公式来看，好像也可以起到表明损失的作用。原因是使用交叉熵的梯度会更大，可以更快的收敛。（在后面会有类似的损失的梯度图）

![image-20200904103824469](.\classification\image-20200904103824469.png)

### （重要）求交叉熵损失函数的微分

对该损失求微分，需要对表达式中的两个与w有关的部分进行求解微分。

在下图中，分别做了这样的工作。

就这样，我们就可以计算出两个偏微分。

![image-20200904104145306](.\classification\image-20200904104145306.png)

计算细节在该图中

先计算w的梯度

![1](.\classification\1.jpg)

再计算b的梯度

![2](.\classification\2.png)

#### 微分在代码中的实现

```python
def gradient(X, Y_label, w, b):
    # This function computes the gradient of cross entropy loss with respect to weight w and bias b.
    y_pred = f(X, w, b)
    pred_error = Y_label - y_pred
    w_grad = -np.sum(pred_error * X.T, 1)
    b_grad = -np.sum(pred_error)
    return w_grad, b_grad
```

```python
w = w - learning_rate / np.sqrt(step) * w_grad
b = b - learning_rate / np.sqrt(step) * b_grad
```

我们将偏微分的结果代入原式，解得原式中的梯度表达式。

整理后，得到如图中的结果。会发现，如果预测值距离偏差值越大，梯度就会越大，趋近的速率就会越快，可见交叉熵是一个比较好用的损失函数。

![image-20200904114145587](.\classification\image-20200904114145587.png)

我们通过推导梯度的表达式得出，逻辑回归中和线性回归中的权重更新式子是一样的。

![image-20200904114226186](.\classification\image-20200904114226186.png)

### 为什么不使用Square Error（平方差）

第一个原因，使用平方差的话，梯度如果在“极端近”，“极端远”的时候，都会让梯度变成0。

“极端远”让梯度变成0的情况，并不是我们想要的。

![image-20200904120320468](.\classification\image-20200904120320468.png)

交叉熵和平方差的梯度图。

我们发现交叉熵不会有平方差特有的：“距离越远，反而梯度越小”这个问题。

![image-20200904121013808](.\classification\image-20200904121013808.png)

## 比较两种模型的创造方法

至此，使用梯度下降的方式获取模型，以及通过计算概率的方式获取模型，都已在“逻辑回归”与“盒子模型”中分别进行了阐述。

那么这两种方法究竟谁更好一些呢？

![image-20200904155240105](.\classification\image-20200904155240105.png)

李宏毅教授做了一个测试，在9个维度的输入中，使用梯度下降方式获取的模型具有更高的准确率。

![image-20200904155338326](.\classification\image-20200904155338326.png)

这是使用Generative预测的具体过程。

![image-20200904155528505](.\classification\image-20200904155528505.png)

### Generative Model 的优点

​	只需要较少的训练集就可以完成一次推算。

## 多分类问题

### softmax与crossentropy

我们使用softmax与交叉熵解决多分类的问题。

这里需要注意一下softmax函数的相关特性

![image-20200904170609865](.\classification\image-20200904170609865.png)

求交叉熵，可以反应当前输出与Label的相似程度

![image-20200904171231043](.\classification\image-20200904171231043.png)

### 逻辑回归的局限性与NN的引入

事实上，单独的一次逻辑回归，在解决下列点的分类的过程中是比较困难的。

很难找到一条确切的直线能够将他们区分开来。

![image-20200904171505097](.\classification\image-20200904171505097.png)

但是我们如果事先进行一次额外的处理，例如按下面这个规则进行一次转化后，我们就能够找到一根线，从而将这些数据正确的分类了。

![image-20200904171806932](.\classification\image-20200904171806932.png)

这给我们的启示是，我们可以使用多次逻辑回归，前面的逻辑回归可以用来提取一些特征。

最后一层的逻辑回归再来进行分类。

![image-20200904172013797](.\classification\image-20200904172013797.png)

这样子下来，多次的特征提取，能够让最后分类的效果有更大的提升。

![image-20200904172212506](.\classification\image-20200904172212506.png)

这也就是Neural Network的雏形了。

![image-20200904172554148](.\classification\image-20200904172554148.png)

## 代码

### 运行结果

![image-20200907121359780](.\classification\image-20200907121359780.png)

### 原始代码

```python
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(0)
X_train_fpath = './data/X_train'
Y_train_fpath = './data/Y_train'
X_test_fpath = './data/X_test'
output_fpath = './output_{}.csv'

data_dim: int


def main():
    # Parse csv files to numpy array
    global data_dim
    with open(X_train_fpath) as file:
        next(file)
        x_train = np.array([line.strip('\n').split(',')[1:] for line in file], dtype=float)
    with open(Y_train_fpath) as file:
        next(file)
        y_train = np.array([line.strip('\n').split(',')[1] for line in file], dtype=float)
    with open(X_test_fpath) as file:
        next(file)
        x_test = np.array([line.strip('\n').split(',')[1:] for line in file], dtype=float)

    x_train, x_mean, x_std = normalize(x_train, isTrain=True)
    x_test, _, _ = normalize(x_test, isTrain=False, specified_columns=None, x_mean=x_mean, x_std=x_std)

    dev_ratio = 0.1
    x_train, y_train, x_dev, y_dev = train_dev_split(x_train, y_train, dev_ratio)

    train_size = x_train.shape[0]
    dev_size = x_dev.shape[0]
    test_size = x_test.shape[0]
    data_dim = x_train.shape[1]
    print('Size of training set: {}'.format(train_size))
    print('Size of development set: {}'.format(dev_size))
    print('Size of testing set: {}'.format(test_size))
    print('Dimension of data: {}'.format(data_dim))
    train(x_train, y_train, x_dev, y_dev, train_size, dev_size)


def train(x_train, y_train, x_dev, y_dev, train_size, dev_size):
    global data_dim
    # Zero initialization for weights ans bias
    w = np.zeros((data_dim,))
    b = np.zeros((1,))

    # Some parameters for training
    max_iter = 10
    batch_size = 8
    learning_rate = 0.2

    # Keep the loss and accuracy at every iteration for plotting
    train_loss = []
    dev_loss = []
    train_acc = []
    dev_acc = []

    # Calcuate the number of parameter updates
    step = 1

    # Iterative training
    for epoch in range(max_iter):
        # Random shuffle at the begging of each epoch
        x_train, y_train = shuffle(x_train, y_train)

        # Mini-batch training
        for idx in range(int(np.floor(train_size / batch_size))):
            X = x_train[idx * batch_size:(idx + 1) * batch_size]
            Y = y_train[idx * batch_size:(idx + 1) * batch_size]

            # Compute the gradient
            w_grad, b_grad = gradient(X, Y, w, b)

            # gradient descent update
            # learning rate decay with time
            w = w - learning_rate / np.sqrt(step) * w_grad
            b = b - learning_rate / np.sqrt(step) * b_grad

            step = step + 1

        # Compute loss and accuracy of training set and development set
        y_train_pred = f(x_train, w, b)
        Y_train_pred = np.round(y_train_pred)
        print("Debug")
        print(Y_train_pred)
        print(y_train)
        train_acc.append(accuracy(Y_train_pred, y_train))
        train_loss.append(cross_entropy_loss(y_train_pred, y_train) / train_size)

        y_dev_pred = f(x_dev, w, b)
        Y_dev_pred = np.round(y_dev_pred)
        dev_acc.append(accuracy(Y_dev_pred, y_dev))
        dev_loss.append(cross_entropy_loss(y_dev_pred, y_dev) / dev_size)

    print('Training loss: {}'.format(train_loss[-1]))
    print('Development loss: {}'.format(dev_loss[-1]))
    print('Training accuracy: {}'.format(train_acc[-1]))
    print('Development accuracy: {}'.format(dev_acc[-1]))

    # Loss curve
    plt.plot(train_loss)
    plt.plot(dev_loss)
    plt.title('Loss')
    plt.legend(['train', 'dev'])
    plt.savefig('loss.png')
    plt.show()

    # Accuracy curve
    plt.plot(train_acc)
    plt.plot(dev_acc)
    plt.title('Accuracy')
    plt.legend(['train', 'dev'])
    plt.savefig('acc.png')
    plt.show()


def shuffle(x, y):
    # This function shuffles two equal-length list/array, X and Y, together.
    randomize = np.arange(len(x))
    np.random.shuffle(randomize)
    return x[randomize], y[randomize]


def sigmoid(z):
    # Sigmoid function can be used to calculate probability.
    # To avoid overflow, minimum/maximum output value is set.
    return np.clip(1 / (1.0 + np.exp(-z)), 1e-8, 1 - 1e-8)


def f(x, w, b):
    return sigmoid(np.matmul(x, w) + b)


def accuracy(Y_pred, Y_label):
    # This function calculates prediction accuracy
    acc = 1 - np.mean(np.abs(Y_pred - Y_label))
    return acc


def cross_entropy_loss(y_pred, Y_label):
    cross_entropy = -np.dot(Y_label, np.log(y_pred + 1e-8)) - np.dot((1 - Y_label), np.log(1 - y_pred))
    return cross_entropy


def gradient(X, Y_label, w, b):
    # This function computes the gradient of cross entropy loss with respect to weight w and bias b.
    y_pred = f(X, w, b)
    pred_error = Y_label - y_pred
    w_grad = -np.sum(pred_error * X.T, 1)
    b_grad = -np.sum(pred_error)
    return w_grad, b_grad


def normalize(x: np.ndarray, isTrain=True, specified_columns: np.ndarray = None, x_mean=None, x_std=None):
    if specified_columns is None:
        # 如果没有指定columns，那么所有的都将包括在这个范围内
        specified_columns = np.arange(x.shape[1])
    if isTrain:
        x_mean = np.mean(x[:, specified_columns], 0).reshape(1, -1)
        x_std = np.std(x[:, specified_columns], 0).reshape(1, -1)
    x[:, specified_columns] = (x[:, specified_columns] - x_mean) / (x_std + 1e-8)
    return x, x_mean, x_std


def train_dev_split(X, Y, dev_ratio=0.25):
    # This function spilts data into training set and development set.
    train_size = int(len(X) * (1 - dev_ratio))
    return X[:train_size], Y[:train_size], X[train_size:], Y[train_size:]


if __name__ == '__main__':
    main()
```

