# CGAN

Conditional Generative Adversarial Nets 有条件的生成对抗网络

相较于原始的GAN的无序性来说，CGAN具有更强的可控性：即，通过标签，生成指定类型的对象。

## 如何工作

 在原始GAN的基础上，让C的信息对Discriminator与Generator产生影响

![image-20200716215601098](.\dp_8_1.png)

## 模块介绍

### tf.concat 拼接向量

```python
t1 = [[1, 2, 3], [4, 5, 6]]
t2 = [[7, 8, 9], [10, 11, 12]]
#按照第0维连接
tf.concat( [t1, t2]，0) ==> [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]]
#按照第1维连接
tf.concat([t1, t2]，1) ==> [[1, 2, 3, 7, 8, 9], [4, 5, 6, 10, 11, 12]]
```

## Coding

### 导入依赖

```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

import tensorflow as tf
import tensorflow.contrib.slim as slim
import tensorflow.examples.tutorials.mnist.input_data as input_data

import numpy as np
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec

tf.set_random_seed(2017)

plt.rcParams['figure.figsize'] = (10.0, 8.0)  # 设置画图的尺寸
plt.rcParams['image.interpolation'] = 'nearest'
plt.rcParams['image.cmap'] = 'gray'


def show_images(images):  # 定义画图工具
    images = np.reshape(images, [images.shape[0], -1])
    sqrtn = int(np.ceil(np.sqrt(images.shape[0])))
    sqrtimg = int(np.ceil(np.sqrt(images.shape[1])))

    fig = plt.figure(figsize=(sqrtn, sqrtn))
    gs = gridspec.GridSpec(sqrtn, sqrtn)
    gs.update(wspace=0.05, hspace=0.05)
    for i, img in enumerate(images):
        ax = plt.subplot(gs[i])
        plt.axis('off')
        ax.set_xticklabels([])
        ax.set_yticklabels([])
        ax.set_aspect('equal')
        plt.imshow(img.reshape([sqrtimg, sqrtimg]))

    return


def deprocess_img(x):
    return (x + 1.0) / 2.0
```

### 导入数据集

注意这里**使用了one_hot**!!

```python
# mnist = input_data.read_data_sets('MINST_data')
mnist = input_data.read_data_sets('MNIST_data', one_hot=True)
train_set = mnist.train
test_set = mnist.test
```

如果不使用one_hot

```python
# 不使用one_hot的情况下 train_labels.shape = (batch x 1)
# 使用了one_hot之后 train_labels.shape = (batch x 10)
train_imgs, train_labels = train_set.next_batch(batch)
```

### 定义holder

```python
input_ph = tf.placeholder(tf.float32, shape=[None, 784], name="inputdata")
label_ph = tf.placeholder(tf.float32, shape=[None, 10])
# 为什么inputs需要经过这种运算呢?
inputs = tf.divide(input_ph - 0.5, 0.5)
```

### 判别器与生成器

```python
def discriminator(data, scope="discriminator", reuse=None):
    with tf.variable_scope(scope, reuse=reuse):
        with slim.arg_scope([slim.fully_connected], activation_fn=None):
            # data 的矩阵形式为 (batch_size x 784)
            net = slim.fully_connected(data, 256, scope='fc1')
            net = tf.nn.leaky_relu(net, alpha=0.2, name='act1')
            net = slim.fully_connected(net, 256, scope='fc2')
            net = tf.nn.leaky_relu(net, alpha=0.2, name='act2')
            net = slim.fully_connected(net, 1, scope='fc3')
            # 经过网络之后 (batch_size x 1)
            # 数据个数，即行数，不变
            return net


def generator(noise, scope='generator', reuse=None):
    with tf.variable_scope(scope, reuse=reuse):
        with slim.arg_scope([slim.fully_connected], activation_fn=tf.nn.relu):
            # noise 是一个 (batch_size x noise_dim)大小的矩阵
            net = slim.fully_connected(noise, 1024, scope='fc1')
            # 经过一次网络之后，变为 (batch_size x 1024)
            net = slim.fully_connected(net, 1024, scope='fc2')
            # 再经过一次网络，变为 (batch_size x 1024)
            net = slim.fully_connected(net, 784, activation_fn=tf.tanh, scope='fc3')
            # 最后经过网络，变为 (batch_size x 784)。并且使用tanh对数据进行裁剪到-1到1
            # 我们发现它的行数（数据个数）是不会发生改变的
            return net
```

### GAN所需要的正反label

```python
batch_size = tf.shape(input_ph)[0]
# true_label与fake_label 都成为(batch_size x 1)的列向量，数值全为 1 （fake_label是0）
true_labels = tf.ones((batch_size, 1), dtype=tf.int64, name='true_labels')
fake_labels = tf.zeros((batch_size, 1), dtype=tf.int64, name='fake_labels')
```

### 定义判别器差别函数

```python
def discriminator_loss(logics_real, logics_fake, scope='D_Loss'):
    with tf.variable_scope(scope):
        loss = tf.losses.log_loss(true_labels, tf.sigmoid(logics_real)) + tf.losses.log_loss(fake_labels,tf.sigmoid(logics_fake))
        return loss


def generator_loss(fake, scope='G_loss'):  # 生成网络的`loss`
    with tf.variable_scope(scope):
        loss = tf.losses.log_loss(true_labels, tf.sigmoid(fake))
        return loss
```

### 搭建网络

```python
noise_dim = 96
# (batch_size x noise_dim)的矩阵，数值为(-1,1)均匀分布
sample_noise = tf.random_uniform([batch_size, noise_dim], dtype=tf.float32, minval=-1.0, maxval=1.0,
                                 name='sample_noise')
# (batch_size x 784)
inputs_fake = generator(tf.concat([sample_noise, label_ph], 1))
# (batch_size x 1)
logits_real = discriminator(tf.concat([inputs, label_ph], 1))
# (batch_size x 1)
logits_fake = discriminator(tf.concat([inputs_fake, label_ph], 1), reuse=True)
```

### 使用判别函数进行优化

```python
# 分别求出两个网络的损失函数，损失函数的构造逻辑可以仔细分析
d_total_error = discriminator_loss(logits_real, logits_fake)
g_total_error = generator_loss(logits_fake)

# 构建优化器
opt = tf.train.AdamOptimizer(3e-4, beta1=0.5, beta2=0.999)
# 为什么此处的优化器中，必须指定var_list呢？
discriminator_params = tf.trainable_variables('discriminator')
train_discriminator = opt.minimize(d_total_error, var_list=discriminator_params)

# generator 训练之前，必须先训练discriminator 所以有一个control_dependencies
generator_params = tf.trainable_variables('generator')
with tf.control_dependencies([train_discriminator]):
    train_generator = opt.minimize(g_total_error, var_list=generator_params)
```

### 开始优化与模型保存

```python
sess = tf.Session()
sess.run(tf.global_variables_initializer())

iter_count = 0
show_every = 1000

# 最多完整训练十次训练集
for e in range(10):
    num_examples = 0
    # 训练一次训练集
    while num_examples < train_set.num_examples:
        if num_examples + 128 < train_set.num_examples:
            batch = 128
        else:
            batch = train_set.num_examples - num_examples
        num_examples += batch
        train_imgs, train_labels = train_set.next_batch(batch)
        # print(train_labels.shape)
        # print(train_imgs.shape)
        loss_d, loss_g, fake_imgs, train_labels = sess.run([d_total_error, g_total_error, inputs_fake, train_generator],
                                                           feed_dict={input_ph: train_imgs, label_ph: train_labels})
        if iter_count % show_every == 0:
            print('Iter: {},D: {:.4f},G:{:.4f}'.format(iter_count, loss_d, loss_g))
            imgs_numpy = deprocess_img(fake_imgs)
            show_images(imgs_numpy[:16])
            plt.show()

            # 查看一下原图
            # print("show img")
            # imgs_numpy = deprocess_img(train_imgs)
            # show_images(imgs_numpy[:16])
            # plt.show()
            # print()

        iter_count += 1

saver = tf.train.Saver()
saver.save(sess, "D:\\Storage\\pycharmProjects\\demo\\gan\\model\\cgan.ckpt")
g = tf.get_default_graph()
# 获取图文件的命令 这里我使用了绝对路径
tf.summary.FileWriter('D:\\Storage\\pycharmProjects\\demo\\graph', g)
```

