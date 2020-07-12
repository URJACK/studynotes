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

