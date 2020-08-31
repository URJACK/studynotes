# 多层神经网络

## 前言

这其实就是一个最简单的单层神经网络

![image-20200715110313919](D:\LearningNotes\ai\deeplearning\dp_7_1.png)

实际上多层神经网络的连接，高层神经网络能够提取更为高级的特征。

## 激活函数

常见的激活函数如下

![image-20200715110613696](D:\LearningNotes\ai\deeplearning\dp_7_2.png)

### 为什么需要激活函数？

对于一个两层的神经网络来说
$$
y=w_2A(w_1x)
$$
如果没有激活函数的话，那么
$$
y=w_2(w_1x) =w^{1,2}x
$$
最终就等价为一个单层的神经网络了

## 两层神经网络

来尝试解决之前单层神经网络逻辑回归难以解决的分类问题

![dp_5_1](.\dp_5_1.png)

**生成数据集的模块**与**绘图模块**见之前的**deeplearning_5_nn2**

### 定义神经网络所需要的参数

```python
# 构造神经网络的参数
with tf.variable_scope('layer_1'):
    # 注意这里w1与b1的shape，他们不再是shape=(2,1)直接到输出了，而是shape(2,4)，先将二个特征转为四个特征
    w1 = tf.get_variable(initializer=tf.random_normal_initializer(stddev=0.01), shape=(2, 4), name="weights1")
    b1 = tf.get_variable(initializer=tf.zeros_initializer(), shape=4, name="biase1")

with tf.variable_scope('layer_2'):
    # 将第一层的四个特征转化为输出的一个特征
    # 不难发现b的维度，始终与w的输出特征个数是相同的
    w2 = tf.get_variable(initializer=tf.random_normal_initializer(stddev=0.01), shape=(4, 1), name="weights2")
    b2 = tf.get_variable(initializer=tf.zeros_initializer(), shape=1, name="biase2")
```

### 定义一个两层神经网络

```python
def two_network(nn_input):
    with tf.variable_scope('two_network'):
        net = tf.matmul(nn_input, w1) + b1
        net = tf.tanh(net)
        net = tf.matmul(net, w2) + b2
        return tf.sigmoid(net)


y_ = two_network(x)
loss = tf.losses.log_loss(predictions=y_, labels=y, scope='loss')

lr = 1
optimizer = tf.train.GradientDescentOptimizer(learning_rate=lr)
train_op = optimizer.minimize(loss=loss, var_list=[w1, w2, b1, b2])
```

此神经网络的表达式等于
$$
y=w_2tanh(w_1x +b_1)+b_2
$$

### 训练神经网络

```python
sess = tf.InteractiveSession()
sess.run(tf.global_variables_initializer())

for e in range(10000):
    sess.run(train_op)
    if (e + 1) % 1000 == 0:
        loss_numpy = sess.run(loss)
        print('训练 %d 次 损失 %0.8f' % (e, loss_numpy))

saver = tf.train.Saver()
saver.save(sess, "D:\\Storage\\pycharmProjects\\demo\\nn\\model\\mnn.ckpt")
```

### 查看运行效果

```python
mnn_input = tf.placeholder(shape=(None, 2), dtype=tf.float32, name="mnn_input")
mnn_output = two_network(mnn_input)


def plot_mnn(x_data):
    y_pred_numpy = sess.run(mnn_output, feed_dict={mnn_input: x_data})
    out = np.greater(y_pred_numpy, 0.5).astype(np.float32)
    return np.squeeze(out)


# 调用显示分布图的函数
plot_decision_boundary(plot_mnn, sess.run(x), sess.run(y))
```

![image-20200715174731150](.\dp_7_3.png)



发现运行效果比之前单层的神经网络就好了太多了

## 多层神经网络

绘图函数、数据集生成函数不变，与之前相同

### 网络定义

```python
def hidden_layer(layer_input, output_depth, scope='hidden_layer', reuse=None):
    # 定义隐藏层
    input_depth = layer_input.get_shape()[-1]
    with tf.variable_scope(scope, reuse=reuse):
        w = tf.get_variable(initializer=tf.random_normal_initializer(), shape=(input_depth, output_depth),
                            name='weights')
        b = tf.get_variable(initializer=tf.zeros_initializer(), shape=output_depth, name='biase')
        net = tf.matmul(layer_input, w) + b
        return net


def DNN(x, net_depths, scope='DNN', reuse=None):
    net = x
    for i, net_depth in enumerate(net_depths):
        net = hidden_layer(net, net_depth, 'layer%d' % i, reuse=reuse)
        net = tf.tanh(net)
    net = hidden_layer(net, 1, scope='classification', reuse=reuse)
    net = tf.sigmoid(net)
    return net


#第一次使用这个网络的时候，reuse默认是None，但后续使用的时候，reuse需要更改为True
y_ = DNN(x, [10, 10, 10, 10])

loss = tf.losses.log_loss(predictions=y_, labels=y)

lr = 1e-1
optimizer = tf.train.GradientDescentOptimizer(learning_rate=lr)
train_op = optimizer.minimize(loss)
```

### 查看训练效果

```python
mnn_input = tf.placeholder(shape=(None, 2), dtype=tf.float32, name="mnn_input")
mnn_output = DNN(mnn_input, [10, 10, 10, 10], reuse=True)


def plot_mnn(x_data):
    y_pred_numpy = sess.run(mnn_output, feed_dict={mnn_input: x_data})
    out = np.greater(y_pred_numpy, 0.5).astype(np.float32)
    return np.squeeze(out)


# 调用显示分布图的函数
plot_decision_boundary(plot_mnn, sess.run(x), sess.run(y))
```

![image-20200715181620420](D:\LearningNotes\ai\deeplearning\dp_7_4.png)

与两层简单的结构相比似乎发现了一些更潜在的可能性。