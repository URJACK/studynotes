# 神经网络_2

紧接上一部分的神经网络

## 逻辑回归

| 时间 | 成绩 | 结果   |
| ---- | ---- | ------ |
| 2    | 3    | 不通过 |
| 3    | 5    | 不通过 |
| 4    | 8    | 通过   |
| 5    | ？   | ？     |

$$
y^{esti}_i=\sigma(wx_i+b)
$$

$$
\sigma(x) = sigmoid(x) = 1 / (1 + e^{-x})
$$

比如可以约定sigmoid(x)>0.7，判定为true，sigmoid(x)<0.7，判定为false。

具体的阈值，可以自行商定

### 与线性模型的关系

线性模型

```
x -> Linear -> y
```

逻辑回归模型

```
x -> Linear -> Logistic -> y
```

### Loss函数的定义

对于线性回归来说
$$
loss=\sum_{i=1}^{n}(y^{esti}_i-y)^2/n
$$
而对于logistic回归来说
$$
loss = (-1/n)\sum_{i=1}^{n}(y_ilog(y_i^{esti}) +(1-y_i)log(1-y_i^{esti}))
$$
其中，yi是真实的label，只能取0或者1两个值

yi == 1的时候
$$
loss = (-1/n)\sum_{i=1}^{n}(log(y_i^{esti}))
$$

yi == 0的时候

$$
loss = (-1/n)\sum_{i=1}^{n}(log(1-y_i^{esti}))
$$

这是一个非常巧妙的定义，当**yi == 1**的时候，我们为了让loss变小，就必须让**y_estimate**变的更大

反之**yi == 0** 的时候，为了让loss变小，也就必须让**y_estimate**变的更小

### 简化梯度下降的代码

实际上，上一章的求梯度，以及根据梯度、来相减、并且赋值，我们可以用optimizer（优化器）来自动实现

```python
optimizer = tf.train.GradientDescentOptimizer(learning_rate=1, name='optimizer')
train_op = optimizer.minimize(loss)
sess.run(tf.global_variables_initializer())
for i in range(100):
    sess.run(train_op)
    
```

实际上，就连loss函数，也有相当多现成的loss函数，只需要指明需要求loss的两个对象即可

```
loss = tf.losses.log_loss(predictions=y_pred, labels=y_data)
```

### 使用逻辑回归解决一个问题

#### 数据集

我们尝试对这个数据集进行训练，从而能得出，我们放入一个点，它应该是红色还是蓝色。

![image-20200715160548373](.\dp_5_1.png)

首先是这个数据集的生成代码

```python
# Module: Create Image
# x y 就是训练集的 输入与label
np.random.seed(1)
m = 400  # samples size
N = int(m / 2)  # 每一类点的个数
D = 2  # 维度
x = np.zeros((m, D))  # 输入特征(2个维度) 第一个维度是横坐标，第二个维度是纵坐标
y = np.zeros((m, 1), dtype='uint8')  # label 有两种取值（0，1）分别代表红色 和 蓝色
a = 4

for j in range(2):
    ix = range(N * j, N * (j + 1))  # range(0, 200)、range(200, 400) numpy可以使用这种range对象作为索引
    t = np.linspace(j * 3.12, (j + 1) * 3.12, N)  # 生成(0, π) 的数据   t(200,) 是一维向量
    t2 = np.random.randn(N) * 0.2  # 生成标准正态分布数值 因为N==200，所以生成的 t2(200,) 是一个一维向量
    t = t + t2  # theta
    r = a * np.sin(4 * t) + np.random.randn(N) * 0.2  # radius
    x[ix] = np.c_[r * np.sin(t), r * np.cos(t)]  # np.c_代表将两个矩阵横向拼接，可见一维向量非常类似于列向量
    y[ix] = j

# End Module


# 展示训练用的数据集，plot传递的头两个参数都是一维张量
plt.plot(x[:200, 0], x[:200, 1], 'ro', label='red')
plt.plot(x[200:, 0], x[200:, 1], 'bo', label='blue')
plt.show()
```

数据集的生成代码中，注意一些编码技巧：

```
1·
对于输入x、标签y来说，输入x往往具有多列，每一列都是一类特征，而标签y往往是单列。
x与y在行数上往往保持一致。

2·
numpy的一些函数的应用。
linspace、random.randn、sin、cos...
以及x是一个numpy数组x[ix] 传入的ix不仅仅可以是一个数，也可以是一个range对象。
这样可以批量选取x中的元素，从而让“表达式右边的numpy数组的数值”能够快速赋值给x

3·
预先可以使用np.zeros快速生成指定维度的x与y。
```

#### 分布区域绘制函数

此外，为了绘制分布图，我们需要有一个绘制分布图的函数，该函数的核心思想是：

给定的输入(x,y)必定存在x_min,x_max,y_min,y_max。依据这四个参量，我们就可以规划出一个矩阵图。

我们使用一种方法，将矩阵图中均匀分割成一个个的单元格。每一个单元格都有自己的(x,y)数值，我们将这个(x,y)代入之前训练好的模型中进行判定，得出z = model(x,y)。 z的取值，就是分布图单个元素的颜色。

```python
# Module show Image

def plot_decision_boundary(model, x, y):
    x_min, x_max = x[:, 0].min() - 1, x[:, 0].max() + 1
    y_min, y_max = x[:, 1].min() - 1, x[:, 1].max() + 1
    # 绘图网格的单位长度
    h = 0.01
    # 将生成的x向量与y向量 扩充为x向量矩阵xx、y向量矩阵yy
    # xx 与 yy具有相同的维度(m x n) 其中xx中，x向量自身是行向量，竖直堆叠
    # yy中，y向量是列向量，横向堆叠
    # 只需要将xx 中每一单元 与 yy 中每一单元 依次对应，就可以得到每一个单元点
    xx, yy = np.meshgrid(np.arange(x_min, x_max, h), np.arange(y_min, y_max, h))
    Z = model(np.c_[xx.ravel(), yy.ravel()])
    # 计算得到的单元点的数值之后，再重新定义矩阵维度
    Z = Z.reshape(xx.shape)
    # 这样，每一个点(xx,yy)，及其点的颜色（函数值Z）都绘制完毕
    plt.contourf(xx, yy, Z, cmap=plt.cm.Spectral)
    plt.xlabel('x1')
    plt.ylabel('x2')
    plt.scatter(x[:, 0], x[:, 1], c=np.squeeze(y), cmap=plt.cm.Spectral)
    plt.show()

    # End Module
```

在上述代码中，meshgrid和ravel函数本身并不复杂，但是它们串联起来的这个逻辑非常重要，详细逻辑如下：

![img](.\dp_5_2.png)

#### 网络的搭建

```python
# 将numpy对象放入tensor中
x = tf.constant(x, dtype=tf.float32, name='x')
y = tf.constant(y, dtype=tf.float32, name='y')

# 使用初始化构造器 获取tensor变量
w = tf.get_variable(initializer=tf.random_normal_initializer(), shape=(2, 1), dtype=tf.float32, name='weights')
b = tf.get_variable(initializer=tf.zeros_initializer(), shape=1, dtype=tf.float32, name='biase')


def logistic_model(inputs):
    logit = tf.matmul(inputs, w) + b
    return tf.sigmoid(logit)


y_ = logistic_model(x)  # 使用这个，搭建神经网络

loss = tf.losses.log_loss(predictions=y_, labels=y)

lr = 1e-1
optimizer = tf.train.GradientDescentOptimizer(learning_rate=lr)
train_op = optimizer.minimize(loss)

sess = tf.InteractiveSession()
sess.run(tf.global_variables_initializer())

TARINING_TIME = 1000
for e in range(TARINING_TIME):
    sess.run(train_op)
    if e % 100 == 0:
        loss_numpy = sess.run(loss)
        print("训练 %d 次， 损失 %.12f " % (e + 1, loss_numpy))


# 定义一个输入接口
data_input = tf.placeholder(shape=(None, 2), dtype=tf.float32, name="logistic_input")
logistic_output = logistic_model(data_input)


def plot_logistic(x_data):
    y_pred_numpy = sess.run(logistic_output, feed_dict={data_input: x_data})
    out = np.greater(y_pred_numpy, 0.5).astype(np.float32)
    return np.squeeze(out)


# 调用显示分布图的函数
plot_decision_boundary(plot_logistic, sess.run(x), sess.run(y))
```

#### 实现效果

![image-20200715164833316](.\dp_5_3.png)

显然我们发现，这种训练效果并不是我们所需要的，这样的训练效果显然无法满足我们的需求。

至于如何进行更好效果的训练，请参考**多层神经网络**。