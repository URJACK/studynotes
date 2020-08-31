---
typora-copy-images-to: ./
---

# 生成对抗网络

训练一个网络，这个网络的输出与输入能够尽可能的接近。

## 从输入到“输入”----自动编码器(AE)

### 构建思路

$$
photo_{input} -> NN_{encoder} ->code->NN_{decoder}->photo_{output}
$$

只取解码器的部分(code -> NN -> photo)

就可以从一个code中，自动生成一张图片。（相当于是一次图片的有损压缩）

### 代码实现

$$
photo_{input}(28*28)->128->64->12->3->12->64->128->photo_{output}(28*28)
$$

![dp_6_1](.\dp_6_1.png)

损失函数如下

```
loss = tf.losses.mean_squared_error(inputs,output)
opt = tf.train.AdamOptimizer(1e-3)
opt.minimize(loss)
```

![dp_6_2](.\dp_6_2.png)

借助工具，让之前浓缩到三维的向量可视化。

经过一次网络的搭建，我们就得到了自动编码器，使用其解码器部分就可以得出图片，但是这样的图片可能是不清晰的。

![image-20200710161802153](.\dp_6_3.png)

以下是新的代码

![dp_6_4](.\dp_6_4.png)

### 总结

综上，得到autoencoder后，我们可以通过code生成一张图片，但是取什么样的code，生成什么样的图片，这些是事先不可知的。

## 变分自动编码器(VAE)

### 引言

为什么引入变分自动编码器？因为自动编码器中，code的分布实在是难以找到规律。

所以我们想要让code的分布尽可能的趋近正态分布。

![dp_6_5](.\dp_6_5.png)

![dp_6_6](.\dp_6_6.png)

## 生成对抗网络（GAN）

### 引言

GAN由两部分组成

1. 生成网络
2. 判别网络

生成网络是利用随机噪声生成“假图片”，而判别网络对真实图片和生成的“假图片”进行真伪判别。

判别网络：是一个简单的“真，假”分类器，只能够判别图片是否真实，而与原来图片有多少分类无关。

生成网络：其构造与自动编码器的解码器类似，从而能够从一个随机噪音中，取得一张图片。

### 对抗学习

1. 优化生成：

   尽量优化生成网络，增大“假图片”识别为“真图片”的概率。

2. 优化判别：

   随后，随着假图片越来越真，判别网络无法识别真图片和假图片的时候，必须优化判别网络，使其能够识别真伪。

3. 再次优化生成：

   然后套娃，循环往复。
   

这个就像是判别网络与生成网络在互相博弈，在学习中不断优化，故称之为**对抗学习**。

### 构造网络

训练之前

![image-20200713095307843](.\dp_6_9.png)

训练结果的如下

![dp_6_7](.\dp_6_7.png)

被训练图片如下

![image-20200713095058239](.\dp_6_8.png)

可以看出针对图片的时候，即是多次训练，效果也未必能有太大的好转

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


mnist = input_data.read_data_sets('MINST_data')
train_set = mnist.train
test_set = mnist.test

# input_ph 一个输入占位符，在数据集较大的时候，每次的输入可以是一部分
# sess.run([d_total_error, g_total_error, inputs_fake, train_generator],feed_dict={input_ph: train_imgs})
# 之后操作feed_dict来填充这种占位符
# 784是图片的像素大小(28 x 28) 而None之所以没有指定是因为可以通过feed_dict实际填充的数据来指定
input_ph = tf.placeholder(tf.float32, shape=[None, 784])
# 为什么inputs需要经过这种运算呢?
inputs = tf.divide(input_ph - 0.5, 0.5)


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


# 同样，batch_size尽管是一个“数”，但是因为input_ph只是一个占位符，实际元素还没有被填充，所以只能是一个暂定对象
# batch_size的意义是：代表每次局部从数据集中抽取的 独立数据样本的个数
batch_size = tf.shape(input_ph)[0]
# true_label与fake_label 都成为(batch_size x 1)的列向量，数值全为 1 （fake_label是0）
true_labels = tf.ones((batch_size, 1), dtype=tf.int64, name='true_labels')
fake_labels = tf.zeros((batch_size, 1), dtype=tf.int64, name='fake_labels')


def discriminator_loss(logics_real, logics_fake, scope='D_Loss'):
    with tf.variable_scope(scope):
        loss = tf.losses.log_loss(true_labels, tf.sigmoid(logics_real)) + tf.losses.log_loss(fake_labels,
                                                                                             tf.sigmoid(logics_fake))
        return loss


def generator_loss(fake, scope='G_loss'):  # 生成网络的`loss`
    with tf.variable_scope(scope):
        loss = tf.losses.log_loss(true_labels, tf.sigmoid(fake))
        return loss


noise_dim = 96
# (batch_size x noise_dim)的矩阵，数值为(-1,1)均匀分布
sample_noise = tf.random_uniform([batch_size, noise_dim], dtype=tf.float32, minval=-1.0, maxval=1.0,
                                 name='sample_noise')
# (batch_size x 784)
inputs_fake = generator(sample_noise)
# (batch_size x 1)
logits_real = discriminator(inputs)
# (batch_size x 1)
logits_fake = discriminator(inputs_fake, reuse=True)

# logits_real 与 logits 都是(batch_size x 1)的矩阵
# discriminator_loss()函数内部的true_labels 与 fake_labels 也都是(batch_size x 1)的矩阵
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

sess = tf.Session()
sess.run(tf.global_variables_initializer())

iter_count = 0
show_every = 1000

# 最多完整训练十次训练集
for e in range(10):
    num_examples = 0
    # 训练一次训练集
    while num_examples < train_set.num_examples:
        # 通过打印信息，我们发现，这是一个循环调用训练集的过程
        # next_batch函数自身就就可以循环调用训练集
        # print("num_examples:", num_examples)
        # print("train_set.num_examples", train_set.num_examples)
        if num_examples + 128 < train_set.num_examples:
            batch = 128
        else:
            batch = train_set.num_examples - num_examples
        num_examples += batch
        # 前文的所有batch_size，都是在这里指定的
        train_imgs, _ = train_set.next_batch(batch)
        loss_d, loss_g, fake_imgs, _ = sess.run([d_total_error, g_total_error, inputs_fake, train_generator],
                                                feed_dict={input_ph: train_imgs})
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

```

### 深度卷积对抗神经网络(DCGAN)

![dp_6_13](.\dp_6_13.png)

仅仅1000次

![image-20200713103449416](.\dp_6_10.png)

4000次

![image-20200713103719840](.\dp_6_11.png)

相较于之前的网络，构造代码仅需要更改两处

第一处是判别器与生成器采用了卷积的方式

```python
def dc_discriminator(data, scope="dc_discriminator", reuse=None):
    with tf.variable_scope(scope, reuse=reuse):
        with slim.arg_scope([slim.conv2d, slim.fully_connected], activation_fn=None):
            net = slim.conv2d(data, 32, 5, stride=1, scope='conv1')
            net = tf.nn.leaky_relu(net, alpha=0.2, name='act1')
            net = slim.max_pool2d(net, 2, stride=2, scope='maxpool1')
            net = slim.conv2d(net, 64, 5, stride=1, scope='conv2')
            net = tf.nn.leaky_relu(net, alpha=0.2, name='act2')
            net = slim.max_pool2d(net, 2, stride=2, scope='maxpool2')
            net = slim.flatten(net, scope='flatten')
            net = slim.fully_connected(net, 1024, scope='fc3')
            net = tf.nn.leaky_relu(net, alpha=0.01, name='act3')
            net = slim.fully_connected(net, 1, scope='fc4')
            return net


def dc_generator(noise, scope='dc_generator', reuse=None):
    with tf.variable_scope(scope, reuse=reuse):
        with slim.arg_scope([slim.fully_connected, slim.conv2d_transpose], activation_fn=None):
            net = slim.fully_connected(noise, 1024, scope='fc1')
            net = tf.nn.relu(net, name='act1')
            net = slim.batch_norm(net, scope='bn1')
            net = slim.fully_connected(net, 7 * 7 * 128, scope='fc2')
            net = tf.nn.relu(net, name='act2')
            net = slim.batch_norm(net, scope='bn2')
            net = tf.reshape(net, (-1, 7, 7, 128))
            net = slim.conv2d_transpose(net, 64, 4, stride=2, scope='convT3')
            net = tf.nn.relu(net, name='act3')
            net = slim.batch_norm(net, scope='bn3')
            net = slim.conv2d_transpose(net, 1, 4, stride=2, scope='convT4')
            net = tf.tanh(net, name='tanh')
            return net
```

第二处是，传入参数给判别器和生成器之前，需要进行一次reshape

```python
dc_inputs = tf.reshape(inputs, (-1, 28, 28, 1))

inputs_fake = dc_generator(sample_noise)
logits_real = dc_discriminator(dc_inputs)
logits_fake = dc_discriminator(inputs_fake, reuse=True)
```