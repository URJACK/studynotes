# 卷积神经网络

## 常见操作

### 数据预处理

数据预处理最常⻅的方法就是**中心化**和**标准化**，**中心化**相当于修正数据的中心位置，实现方法非常简单，就是在每个特征维度上减去对应的均值，最后得到 0 均值的特征。**标准化**也非常简单，在数据变成 0 均值之后，为了使得不同的特征维度有着相同的规模，可以除以标准差近似为一个标准正态分布，也可以依据最大值和最小值将其转化为 -1 ~ 1 之间

![image-20200723112105089](.\imgs_11\1.png)

均值
$$
u_b=\frac{\sum_{i=1}^{m}x_i}{m}
$$
方差
$$
\sigma_b^2=\frac{\sum^m_{i=1}(x_i-u_b)^2}{m}
$$
处理后的x
$$
x_i^{e}=\frac{x_i-u_b}{\sqrt[2]{\sigma_b^2+\epsilon}}
$$
输出结果
$$
y_i^e=\gamma x_i^e+\beta
$$
Coding

```python
# 1
tf.contrib.layers.batch_norm 是一个内置的批标准化函数

# 2
import tensorflow.contrib.slim as slim
slim.batch_norm 也是一个标准化函数
```



## AlexNet

### 介绍

![image-20200724092324377](.\imgs_11\2.png)

![image-20200724092814261](.\imgs_11\image-20200724092814261.png)