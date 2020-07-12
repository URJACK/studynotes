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

![dp_6_1](D:\LearningNotes\ai\deeplearning\dp_6_1.png)

损失函数如下

```
loss = tf.losses.mean_squared_error(inputs,output)
opt = tf.train.AdamOptimizer(1e-3)
opt.minimize(loss)
```

![dp_6_2](D:\LearningNotes\ai\deeplearning\dp_6_2.png)

借助工具，让之前浓缩到三维的向量可视化。

经过一次网络的搭建，我们就得到了自动编码器，使用其解码器部分就可以得出图片，但是这样的图片可能是不清晰的。

![image-20200710161802153](D:\LearningNotes\ai\deeplearning\dp_6_3.png)

以下是新的代码

![dp_6_4](D:\LearningNotes\ai\deeplearning\dp_6_4.png)

### 总结

综上，得到autoencoder后，我们可以通过code生成一张图片，但是取什么样的code，生成什么样的图片，这些是事先不可知的。

## 变分自动编码器(VAE)

### 引言

为什么引入变分自动编码器？因为自动编码器中，code的分布实在是难以找到规律。

所以我们想要让code的分布尽可能的趋近正态分布。

![dp_6_5](D:\LearningNotes\ai\deeplearning\dp_6_5.png)

![dp_6_6](D:\LearningNotes\ai\deeplearning\dp_6_6.png)

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

**判别网络**是一个二分器：

![dp_6_7](D:\LearningNotes\ai\deeplearning\dp_6_7.png)

**生成网络**是一个根据随机噪音生成数据的过程

![dp_6_8](D:\LearningNotes\ai\deeplearning\dp_6_8.png)

**判别网络**的**损失函数**：“真”的数据尽量判别为真、“假”的数据尽量判别为假

![dp_6_9](D:\LearningNotes\ai\deeplearning\dp_6_9.png)

**生成网络**的**损失函数**："假"的数据，尽量判别为真

![dp_6_10](D:\LearningNotes\ai\deeplearning\dp_6_10.png)

构建对抗学习

![image-20200710235138709](D:\LearningNotes\ai\deeplearning\dp_6_11.png)

开始训练

![dp_6_12](D:\LearningNotes\ai\deeplearning\dp_6_12.png)

### 深度卷积对抗神经网络(DCGAN)

![dp_6_13](D:\LearningNotes\ai\deeplearning\dp_6_13.png)