# 神经网络

## 常见的学习方式

### 监督学习

#### 定义

对已经标记的训练数据样本进行学习，然后对样本外的数据进行预测。（典型应用：分类垃圾邮件）

### 非监督学习

对没有标记的训练样本进行学习，然后发现其中结构性的知识。（典型应用：聚类商店顾客，A类顾客喜欢珠宝、B类顾客喜欢钱包）

### 强化学习

不断依据环境做决策，然后环境根据决策进行惩罚或者奖励。

## 线性模型

### 线性模型是什么

使用线性模型，预测可以得到结果

<table>
	<tr>
		<td>时间</td>
	    <td>成绩</td>
	    <td> </td>
	</tr>
	<tr>
	    <td>2</td>
	    <td>3</td>
        <td rowspan="3">训练集</td>
	</tr>
	<tr>
	    <td>3</td>
	    <td>5</td>
	</tr>
	<tr>
	    <td>4</td>
	    <td>8</td>
	</tr>
    <tr>
    	<td>5</td>
    	<td>?</td>
        <td>测试集</td>
    </tr>
</table>
线性模型的公式：
$$
y_{expected} = wx_i+b
$$

预测值与真实值存在着差异，称为loss

$$
loss_i=(yi_{expected}-yi)^2
$$

这是平均loss

$$
loss_{ave}=(1/n)\sum_{i=1}^{n}(yi_{expected}-yi)^2
$$

以下是平均loss和loss在一组数据中的情况

| 时间 | y    | 预测 | loss |
| ---- | ---- | ---- | ---- |
| 1    | 1.2  | 0.5  | 0.49 |
| 2 | 3 | 2 | 1    |
| 3 | 5 | 3.5 | 2.25 |
| 4 | 8 | 6 | 4    |
|      |      |          |loss的平均值=1.9|

### 梯度下降

为了调整线性模型的参数，让它的loss平均值更小，此处引入梯度下降法。

#### 梯度

梯度就是导数，比如
$$
f(x)= x^2
$$
在x = 1的时候，梯度就是2

对于更为一般的情况，如果一个函数f(x,y)有多个变量
$$
(\delta f/\delta x,\delta f/\delta y)
$$
梯度即为f分别对x与y求偏导

#### 梯度下降的具体实现

Θ是我们定义的学习率。学习率越大，就能更快的趋近loss的最小值，但是如果太大，可能会让参数变化过于明显，从而导致一些反效果，所以学习率调整，是非常关键的。

按照这种方式来迭代每次w的数值，可以让loss越来越小
$$
w = w - \theta (\delta l/\delta w)
$$
问题来了，为什么每次按照梯度进行这样的变化会让loss越来越小呢？

原因是
$$
loss = (wx + b - y)^2
$$
y本来就是一个确定值，是常量。在对w求偏导的时候，b也是看作常量。

综上，对w求偏导的时候，loss(x)是一个开口向上的一元二次函数。

所以该函数上的任意一点，对梯度进行负变化的时候，都会使得loss的值变小。

#### 代码

```
sess = tf.InteractiveSession()

# 定义x，y两个变量
x = tf.Variable(initial_value=[1, 2, 3, 4], dtype=tf.float32, name='x')
y = tf.Variable(initial_value=[1.2, 3, 5, 8], dtype=tf.float32, name='y')

# weight权重设置为随机数，但是biase偏差量我们设置为0
w = tf.Variable(initial_value=tf.random_normal(shape=(), seed=2019,name="randomWeight"), dtype=tf.float32, name='weight')
b = tf.Variable(initial_value=0, dtype=tf.float32, name='biase')

# 声明一个线性模型作用域，因为这里有太多的运算，这样整体可以看成是一个模块
with tf.variable_scope('Linear_Model'):
    y_pred = w * x + b

# 为什么这里要调用一次reduce_mean呢？
# reduce_mean:计算向量的各个维度上的元素的平均值.
# 因为y可能不是单独的一个数值，而是一个向量，同样x也可能是一个向量
# 虽然 w 与 b 是两个变量，即单独的数。但(w * x)也会是一个向量，并且y_pred = (w * x) + b 也会是一个向量（数与向量相加，变成向量）
# 所以(y - y_pred)也是一个向量，平方运算会对向量上的每一个维度都做平方运算。
# 不过我们最后需要的loss是一个变量值，而不是一个向量，于是调用reduce_mean，将向量的每一个维度，求平均值
loss = tf.reduce_mean(tf.square(y - y_pred))

# 让平均值loss对w与b求导数，得到w和b的梯度
w_grad, b_grad = tf.gradients(loss, [w, b])

# 定义学习率
with tf.variable_scope('Learning_Model'):
    lr = 1e-2
    w_update = w.assign_sub(lr * w_grad)
    b_update = b.assign_sub(lr * b_grad)

# 打印图
tf.summary.FileWriter('D:\\Storage\\pycharmProjects\\demo', tf.get_default_graph())

# 初始化参数
sess.run(tf.global_variables_initializer())

# 运行图
for i in range(10):
    print(sess.run([w_update, b_update]))

sess.close()
```

上述代码是基本实现功能的代码，但是为了更加直观明显的对比训练前后的差距，对上述代码做了一定的增补，详见下文

#### 代码（增添拟真数据和对比信息）

```python
# 导入数据
x_train = np.array(
    [[3.3], [4.4], [5.5], [6.71], [6.93], [4.168], [9.779], [6.182], [7.59], [2.167], [7.042], [10.791], [5.313],
     [7.997], [3.1]], dtype=np.float32)
y_train = np.array(
    [[1.7], [2.76], [2.09], [3.19], [1.694], [1.573], [3.366], [2.596], [2.53], [1.221], [2.827], [3.465], [1.65],
     [2.904], [1.3]], dtype=np.float32)

# 显示初始数据集

# 定义x，y两个变量
x = tf.Variable(initial_value=x_train, dtype=tf.float32, name='x')
y = tf.Variable(initial_value=y_train, dtype=tf.float32, name='y')

# weight权重设置为随机数，但是biase偏差量我们设置为0
w = tf.Variable(initial_value=tf.random_normal(shape=(), seed=2019, name="randomWeight"), dtype=tf.float32,
                name='weight')
b = tf.Variable(initial_value=0, dtype=tf.float32, name='biase')

# 声明一个线性模型作用域，因为这里有太多的运算，这样整体可以看成是一个模块
with tf.variable_scope('Linear_Model'):
    y_pred = w * x + b

# 开启会话与初始化参数
sess = tf.InteractiveSession()
sess.run(tf.global_variables_initializer())

# 可以在这里查看一下最开始的y_pred
y_pred_numpy = y_pred.eval(session=sess)
plt.plot(x_train, y_train, 'bo', label='real')
plt.plot(x_train, y_pred_numpy, 'ro', label='estimated')
plt.show()

# 为什么这里要调用一次reduce_mean呢？
# reduce_mean:计算向量的各个维度上的元素的平均值.
# 因为y可能不是单独的一个数值，而是一个向量，同样x也可能是一个向量
# 虽然 w 与 b 是两个变量，即单独的数。但(w * x)也会是一个向量，并且y_pred = (w * x) + b 也会是一个向量（数与向量相加，变成向量）
# 所以(y - y_pred)也是一个向量，平方运算会对向量上的每一个维度都做平方运算。
# 不过我们最后需要的loss是一个变量值，而不是一个向量，于是调用reduce_mean，将向量的每一个维度，求平均值
loss = tf.reduce_mean(tf.square(y - y_pred))

# 让平均值loss对w与b求导数，得到w和b的梯度
w_grad, b_grad = tf.gradients(loss, [w, b])

# 定义学习率
with tf.variable_scope('Learning_Model'):
    lr = 1e-2
    w_update = w.assign_sub(lr * w_grad)
    b_update = b.assign_sub(lr * b_grad)

# 打印图
# tf.summary.FileWriter('D:\\Storage\\pycharmProjects\\demo', tf.get_default_graph())

# 运行图
for i in range(10):
    print(sess.run([w_update, b_update]))

# 可以在这里查看一下更新的y_pred
y_pred_numpy = y_pred.eval(session=sess)
plt.plot(x_train, y_train, 'bo', label='real')
plt.plot(x_train, y_pred_numpy, 'ro', label='estimated')
plt.show()

sess.close()
```

### 梯度下降-多项式回归

多项式回归（以三项为例）
$$
y_{expected} = w_1x + w_2x^2 + w_3x^3 + b
$$
对比之前的线性回归，这里的x的不同次方，均可以看成是不同的输入数据

之前的w是一个数、而x是一个1维向量： (1x1) * (1xn)

但现在w是一个向量，那么x也必须凑出相应的维度：(1x3) * (3xn)

```python
x_train = np.stack([x_sample ** i for i in range(1, 4)], axis=0)
```

如果w我们设置成（3x1），那么x就必须凑成(nx3)这种维度。最后使用 x * w == (n x 1)

```python
x_train = np.stack([x_sample ** i for i in range(1, 4)], axis=1)
```

```python
import tensorflow.compat.v1 as tf
import numpy as np
import matplotlib.pyplot as plt

tf.disable_eager_execution()

# 模拟设定 w 与 b 的数值、看最后训练出来的w 与 b 能不能接近这两个数值
w_target = np.array([0.5, 3, 2.4])
b_target = np.array([0.9])

# 调试一下表达式
f_des = 'y = {:.2f} + {:.2f} * x + {:.2f} * x^2 + {:.2f} * x^3'.format(b_target[0], w_target[0], w_target[1],
                                                                       w_target[2])
print(f_des)

# 模拟数据集
x_sample = np.arange(-3, 3.1, 0.1)
y_sample = b_target + w_target[0] * x_sample + w_target[1] * x_sample ** 2 + w_target[2] * x_sample ** 3
# plt.plot(x_sample, y_sample, label='real curve')
# plt.show()

# 构建初始数据
x_train = np.stack([x_sample ** i for i in range(1, 4)], axis=1)
x_train = tf.constant(x_train, dtype=tf.float32, name='x_train')
y_train = tf.constant(y_sample, dtype=tf.float32, name='y_train')

w = tf.Variable(initial_value=tf.random_normal(shape=(3, 1), name="random_weight"), dtype=tf.float32, name='weights')
b = tf.Variable(initial_value=0, dtype=tf.float32, name='biase')

# 开启会话与初始化参数
sess = tf.InteractiveSession()
sess.run(tf.global_variables_initializer())

# tf.squeeze之后，默认的，会变成列向量
with tf.name_scope("Linear_Model"):
    y_expected = tf.squeeze(tf.matmul(x_train, w) + b)

# 要使用plot查看一下图也太麻烦了....
# 必须得从tensor类型转化为numpy数组--->  numpy.Array == sess.run(tensor)

y_expected_numpy = sess.run(y_expected)
x_train_numpy = sess.run(x_train)
y_train_numpy = sess.run(y_train)
# x_train_numpy[:,0] 是一个列向量(n x 1) ，数值刚好就是输入的x的数值
plt.plot(x_train_numpy[:, 0], y_train_numpy, 'bo', label='real')
# x_train_numpy[:,1] 与 x_train_numpy[:,2] ，数值刚好是输入的x^2 与 x^3 的数值，这些没有必要作为横坐标
# plt.plot(x_train_numpy[:, 1], y_train_numpy, 'ro', label='real')
# plt.plot(x_train_numpy[:, 2], y_train_numpy, 'go', label='real')
plt.plot(x_train_numpy[:, 0], y_expected_numpy, 'go', label='estimated')
plt.show()

# 求梯度
loss = tf.reduce_mean(tf.square(y_expected - y_train))
w_grad, b_grad = tf.gradients(loss, [w, b])
print("loss", sess.run(loss))
print("w_grad", sess.run(w_grad))
print("b_grad", sess.run(b_grad))

# 学习
with tf.name_scope("Learning_Model"):
    learningRate = 1e-3
    w_update = w.assign_sub(learningRate * w_grad)
    b_update = b.assign_sub(learningRate * b_grad)

for i in range(10):
    print(sess.run([w_update, b_update]))

y_expected_numpy = sess.run(y_expected)
plt.plot(x_train_numpy[:, 0], y_train_numpy, 'bo', label='real')
plt.plot(x_train_numpy[:, 0], y_expected_numpy, 'go', label='estimated')
plt.show()
```

## 总结

1. Tensorflow的理解非常重要
2. 向量的使用（注意多项式回归与单一回归的区别）
3. 还有TensorFlow的一些常见函数的使用

### TensorFlow的流程理解


```
我认为TensorFlow中，除了最开始定义的变量与常量，其它的我们通常在编程语言中认为的中间变量，在它这里都可以认为一种运算，因为所有的这些中间变量，都可以通过最初定义的常变量经过各种运算得到。
每一个运算都具有自己的“输入”、“输出”，分别来自之前的运算、以及输送给之后的运算

唯一实现学习效果的就是assign_sub这个运算，只有带assign的运算，才能确实改变了初始定义的变量的数值
比如 sess.run(assign....)

如果run之前的任意一个运算(Tensor)，都只是重新根据初始定义的常量与变量，经过定义好的Tensor，临时重新计算出来的数值。

这也使得，tensorflow只会运行需要的Tensor，后续的Tensor，不需要进行计算。
```

### 向量的维度

向量的维度处理是一个比较麻烦的事情，需要自己头脑对这些数据的排布具有清晰的认识

```python
x1 = np.arange(9)
print(x1)
print(x1.shape)
```

```
[0 1 2 3 4 5 6 7 8]
(9,)
```

实际上x1可以称作张量，进而与(9x1)列向量与(1x9)行向量做区分

```python
x_sample = np.arange(9)
print(x_sample.shape)
x1 = np.stack([x_sample ** i for i in range(1, 4)], axis=0)
print(x1.shape)
x2 = np.stack([x_sample ** i for i in range(1, 4)], axis=1)
print(x2.shape)
```

```
(9,)
(3, 9)
(9, 3)
```

这里一定要理解axis与stack搭配时的用法，更要理解经过stack之后，数据的结构形式

### TensorFlow一些常见函数

#### Variable

定义变量，通常指明以下三个属性

```
b = tf.Variable(initial_value=0, dtype=tf.float32, name='biase')
```

initial_value 既可以numpy的数组、也可以是tensor

```
# numpy
x_train = np.stack([x_sample ** i for i in range(1, 4)], axis=1)
x = tf.Variable(initial_value=x_train, dtype=tf.float32, name='x')

# Tensor
w = tf.Variable(initial_value=tf.random_normal(shape=(3, 1), name="random_weight"), dtype=tf.float32, name='weights')
```

#### Random

tensorflow中的随机数Tensor需要指定shape.即便是只有一个数也需要制定

```
tf.random_normal(shape=(3, 1), name="random_weight")
tf.random_normal(shape=(), seed=2019, name="randomWeight")
```

#### gratitude

```
# 求梯度常用的两个函数
# 求平均损失,是一个常量
loss = tf.reduce_mean(tf.square(y_expected - y_train))
# 平均损失对w 与 b求导
w_grad, b_grad = tf.gradients(loss, [w, b])
```

