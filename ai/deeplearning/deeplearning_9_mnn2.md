# 多层神经网络

## 多分类问题

多分类问题，仅仅只需要让神经网络的输出层，从原先的0、1变为0~9。

具体实现中，0、1使用sigmoid函数，但0~9中，我们需要使用softmax函数。

### 模块介绍

#### 绘制子图

subplot绘制

![image-20200720222920291](.\dp_9_1.png)

![image-20200720222947123](.\dp_9_2.png)

subplots绘制

![image-20200720223121921](.\dp_9_3.png)

实际上，plot函数需要传入(x,y)来绘制曲线图。

绘制灰度图，可以使用

```python
axes[row][col].imshow(image, cmap='gray')
```

除此之外，还有一些可选的选项

```python
axes[row][col].axis('off') # 不绘制坐标轴
axes[row][col].set_title('%d' % label) # 设置子图的标题
```

#### 交叉熵计算损失函数

```python
# 交叉熵计算损失函数
loss = tf.losses.softmax_cross_entropy(logits=dnn, onehot_labels=label_ph)
```

#### 正确率衡量

##### tf.argmax

axis = 0 的时候，计算每一列最大值的索引 [0 , n)

```
test[0] = array([1, 2, 3])
test[1] = array([2, 3, 4])
test[2] = array([5, 4, 3])
test[3] = array([8, 7, 2])
# output   :    [3, 3, 1]  
```

##### tf.equal

代码

```python
import tensorflow as tf
import numpy as np
 
A = [[1,3,4,5,6]]
B = [[1,3,4,3,2]]
 
with tf.Session() as sess:
    print(sess.run(tf.equal(A, B)))
```

输出

```
[[ True  True  True False False]]
```

##### tf.cast

代码

```python
a = tf.Variable([1,0,0,1,1])
b = tf.cast(a,dtype=tf.bool)
sess = tf.Session()
sess.run(tf.initialize_all_variables())
print(sess.run(b))
```

输出

```
[ True False False  True  True]
```

##### tf.reduce_mean

代码与输出

```python
import tensorflow as tf
 
x = [[1,2,3],
      [1,2,3]]
 
xx = tf.cast(x,tf.float32)
 
mean_all = tf.reduce_mean(xx, keep_dims=False)
mean_0 = tf.reduce_mean(xx, axis=0, keep_dims=False)
mean_1 = tf.reduce_mean(xx, axis=1, keep_dims=False)
 
 
with tf.Session() as sess:
    m_a,m_0,m_1 = sess.run([mean_all, mean_0, mean_1])
 
print m_a    # output: 2.0
print m_0    # output: [ 1.  2.  3.]
print m_1    # output:  [ 2.  2.]
```

![image-20200721092026829](D:\LearningNotes\ai\deeplearning\dp_9_4.png)

### Coding

#### 数据准备、依赖准备

```python
from __future__ import division
from __future__ import absolute_import
from __future__ import print_function

import tensorflow as tf
import numpy as np
import matplotlib.pyplot as plt

import tensorflow.examples.tutorials.mnist.input_data as input_data

tf.set_random_seed(2019)

mnist = input_data.read_data_sets('MNIST_data', one_hot=True)
train_set = mnist.train
test_set = mnist.test
```

#### 绘制初始图片

```python
fig, axes = plt.subplots(ncols=6, nrows=2)
plt.tight_layout(w_pad=-2.0, h_pad=-8.0)

images, labels = train_set.next_batch(12, shuffle=False)

for ind, (image, label) in enumerate(zip(images, labels)):
    image = image.reshape((28, 28))
    label = label.argmax()

    row = ind // 6
    col = ind % 6

    # 填充子图元素
    axes[row][col].imshow(image, cmap='gray')
    axes[row][col].axis('off')
    axes[row][col].set_title('%d' % label)

plt.show()
```

#### 定义隐藏层

```python
def hidden_layer(layer_input, output_depth, scope='hidden_layer', reuse=None):
    input_depth = layer_input.get_shape()[-1]
    with tf.variable_scope(scope, reuse=reuse):
        w = tf.get_variable(initializer=tf.truncated_normal_initializer(stddev=0.1), shape=(input_depth, output_depth),
                            name="weights")
        b = tf.get_variable(initializer=tf.constant_initializer(0.1), shape=output_depth, name='biase')
        net = tf.matmul(layer_input, w) + b
        return net
```

#### 定义深层神经网络

```python
def DNN(x, output_depths, scope='DNN', reuse=None):
    net = x
    for i, output_depth in enumerate(output_depths):
        net = hidden_layer(net, output_depth, scope='layer%d' % i, reuse=reuse)
        net = tf.nn.relu(net)
    net = hidden_layer(net, 10, scope='classification', reuse=reuse)
    # 注意，这里先不加上softmax，就用一个10维输出即可
    return net
```

#### 搭建计算图

```python
# 设置占位符
input_ph = tf.placeholder(shape=(None, 784), dtype=tf.float32)
label_ph = tf.placeholder(shape=(None, 10), dtype=tf.int64)

# 构建一个4层神经网络(400,200,100,10)
dnn = DNN(input_ph, [400, 200, 100])

# 交叉熵计算损失函数
loss = tf.losses.softmax_cross_entropy(logits=dnn, onehot_labels=label_ph)

# 下面定义的是正确率
# 首先取argmax. 因为dnn 是一个(batch_size x 10)的向量。取argmax之后，变为(batch_size)的张量
# 需要注意的是label_ph 也是一个(batch_size x 10)的向量，而不是(batch_size)的张量，所以我们依旧需要取argmax
# 使用equal 对两个张量进行对比，取得(batch_size)张量的True和False
# 使用cast 将equal取得的True False张量转化为(batch_size)张量的浮点数
# 对这个浮点数张量求平均值，即可得到准确率：因为正确的值为1，错误的值为0
acc = tf.reduce_mean(tf.cast(tf.equal(tf.argmax(dnn, axis=-1), tf.argmax(label_ph, axis=-1)), dtype=tf.float32))

# 定义学习率
lr = 0.01
optimizer = tf.train.GradientDescentOptimizer(learning_rate=lr)
train_op = optimizer.minimize(loss)
```

#### 训练开始

```python
sess = tf.InteractiveSession()

batch_size = 64

sess.run(tf.global_variables_initializer())

for e in range(20000):
    images, labels = train_set.next_batch(batch_size)
    sess.run(train_op, feed_dict={input_ph: images, label_ph: labels})
    if e % 1000 == 999:
        # 获取 batch_size 个测试样本
        test_imgs, test_labels = test_set.next_batch(batch_size)
        # 计算在当前样本上的训练以及测试样本的损失值和正确率
        loss_train, acc_train = sess.run([loss, acc], feed_dict={input_ph: images, label_ph: labels})
        loss_test, acc_test = sess.run([loss, acc], feed_dict={input_ph: test_imgs, label_ph: test_labels})
        print('STEP {}: train_loss: {:.6f} train_acc: {:.6f} test_loss: {:.6f}test_acc: {:.6f}'.format(e + 1, loss_train,acc_train,loss_test,acc_test))


print('Train Done!')
print('-' * 30)
```

#### 查看训练效果

```python
test_loss_array = []
test_acc_array = []
for _ in range(test_set.num_examples // 100):
    image, label = test_set.next_batch(100)
    loss_test, acc_test = sess.run([loss, acc], feed_dict={input_ph: image, label_ph: label})
    test_loss_array.append(loss_test)
    test_acc_array.append(acc_test)

print('test loss: {:.6f}'.format(np.array(test_loss_array).mean()))
print('test accuracy: {:.6f}'.format(np.array(test_acc_array).mean()))
```

## 反向传播算法（理论）

### 链式法则

![img](.\dp_9_5.png)

### 优化算法

一个机器学习的问题，都会预先定义一个损失函数，通过最小化这个损失函数，就可以优化模型。即是是一个最大化的问题，在最大化问题前增加一个负号，也可以转化为最小化问题。所以，我们只需要去关注如何最小化一个损失函数。

优化算法的选取极为重要，因为对于复杂的模型而言，损失函数往往没有解析解，必须使用基于数值的方法来找到近似解。即通过不断的迭代来找到最优解。

#### 梯度为0的情况

##### 局部最优解

除了花费时间较长之外，还有可能找到的只是局部最优解，而不是全局最优解。
$$
f(x)=2x-0.5x^2-\frac{2}{3}x^3+{1}+\frac{1}{4}x^4
$$
(-2,3)之间，-1是它的最小值点，2是它的局部最小值点

如果更新到局部最小值点之后，梯度就为0了，陷入该局部最小值点。

##### 鞍点

$$
y=x^3
$$

在x=0的时候，梯度为0，是一个鞍点。因为深度学习复杂的模型，梯度为0的点是鞍点的情况会非常多。

#### 随机梯度下降

$$
\theta^i = \theta^{i-1}-\eta Grad\, L(\theta^{i-1})
$$

$$
\theta^{*}=argmin L(\theta)
$$

##### 泰勒级数

$$
h(x)=\sum_{k=0}^{\infty}\frac{h^{k}(x_0)}{k!}(x - x_0)^k = h(x_0) + h'(x_0)(x-x_0)+\frac{h''(x_0)}{2!}(x-x_0)^2
$$

当x足够接近x0的时候
$$
h(x)\approx h(x_0)+h'(x_0)(x-x_0)
$$
对于多元泰勒级数
$$
h(x,y)\approx h(x_0,y_0)+\frac{\theta h(x_0,y_0)}{\theta x}(x-x_0)+\frac{\theta h(x0,y0)}{\theta y}(y-y_0)
$$
