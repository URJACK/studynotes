# 编码技巧

## numpy相关

### 数据相关

#### 数据形状(shape)

第一个参数是**数据**，第二参数是**形状**

```
all_digits = np.reshape(all_digits, (-1, 28, 28, 1))
```

关于**形状**的注意点：

1. 类型是Tuple。
2. 第一个维度通常不用改变（样本下标）所以直接**填入-1**。

#### 数据的拼接(concatenate)

```python
(x_train, _), (x_test, _) = keras.datasets.mnist.load_data()
all_digits = np.concatenate([x_train, x_test])
print(all_digits.shape)
```

x_train 与 x_test 均为numpy对象，我们对它们使用concatenate

```
(70000, 28, 28, 1)
```

#### 数据类型的暂时变换(astype)

暂时转换自己的数据类型进行运算

```
# .......
all_digits = np.concatenate([x_train, x_test])
all_digits = all_digits.astype("float32") / 255
# .......
```

## tensorflow相关

### 数据相关

#### 数据形状(shape)

```python
tf.shape(real_images)[0]
```

#### 生成指定形状的随机数(random.normal)

```python
tf.random.normal(shape=(batch_size, latent_dim))
```

#### 数据拼接(concat)

```python
labels = tf.concat([tf.ones((batch_size, 1)), tf.zeros((batch_size, 1))], axis=0)
```

### dataset:数据集相关操作

#### 创建数据集(from_tensor_slices,range)

```python
dataset = tf.data.Dataset.from_tensor_slices([1, 2, 3])
dataset = tf.data.Dataset.range(8)
```

#### 遍历数据集(as_numpy_iterator())

打印数据集，我们发现使用from_tensor_slices这个方法

dataset里的元素都会是Tensor对象。**dataset自身就是一个迭代器**，但他还可以使用as_numpy_iterator**返回numpy类型的迭代器**。

```python
dataset = tf.data.Dataset.from_tensor_slices([1, 2, 3])
for element in dataset:
  print(element)
```

---

```
tf.Tensor(1, shape=(), dtype=int32)
tf.Tensor(2, shape=(), dtype=int32)
tf.Tensor(3, shape=(), dtype=int32)
```

如何获取到数据对象（numpy对象）呢？我们需要使用dataset的as_numpy_iterator方法

```python
dataset = tf.data.Dataset.from_tensor_slices([1, 2, 3])
for element in dataset.as_numpy_iterator():
  print(element)
```

---

```
1
2
3
```

#### 数据集分批、打乱、重复(batch)

使用batch对数据集进行分批

```python
dataset = tf.data.Dataset.range(8)
dataset = dataset.batch(3)
for element in dataset.as_numpy_iterator():
    print(element)
for element in dataset:
    print(element)
```

---

```
[0 1 2]
[3 4 5]
[6 7]
tf.Tensor([0 1 2], shape=(3,), dtype=int64)
tf.Tensor([3 4 5], shape=(3,), dtype=int64)
tf.Tensor([6 7], shape=(2,), dtype=int64)
```

使用**shuffle**打乱数据集，使用**repeat**重复数据集

```python
dataset = tf.data.Dataset.range(3)
dataset = dataset.shuffle(3, reshuffle_each_iteration=True)
dataset = dataset.repeat(2)  # doctest: +SKIP
for ele in dataset.as_numpy_iterator():
	print(ele)
```

---

```
2
0
1
2
0
1
```

#### 数据集运算(map)

对数据集里的每一个元素进行**运算**（使用lambda表达式），使用**map**。

```python
def main():
    dataset = tf.data.Dataset.range(5)
    dataset = dataset.map(lambda x: x ** 2)
    for ele in dataset.as_numpy_iterator():
        print(ele)
```

---

```
0
1
4
9
16
```

#### 数据集筛选(filter)

```python
dataset = tf.data.Dataset.from_tensor_slices([1, 2, 3])
dataset = dataset.filter(lambda x: x < 3)
for ele in dataset.as_numpy_iterator():
	print(ele)
```

---

```
1
2
```

#### 数据集拼接(concatenate,zip)

**纵向拼接**，按样本个数拼接，使用**concatenate**。

```python
a = tf.data.Dataset.range(1, 4)  # ==> [ 1, 2, 3 ]
b = tf.data.Dataset.range(4, 8)  # ==> [ 4, 5, 6, 7 ]
ds = a.concatenate(b)
list(ds.as_numpy_iterator())
```

---

```
1
2
3
4
5
6
7
```

**横向拼接**，按样本特征拼接，使用**zip**。

这个与python自带的zip函数很类似

```python
a = tf.data.Dataset.range(1, 4)  # ==> [ 1, 2, 3 ]
b = tf.data.Dataset.range(4, 8)  # ==> [ 4, 5, 6, 7 ]
c = tf.data.Dataset.zip((a, b))
for ele in c.as_numpy_iterator():
    print(ele)
```

---

```
(1, 4)
(2, 5)
(3, 6)
```

## keras相关

### 定义模型

#### 卷积模型的定义

卷积模型中的输入，它的数据维度**必须包含三个维度**（高，宽，通道）

即便是**灰度图**，通道数也必须**设置为1**

```
discriminator = keras.Sequential(
        [
            keras.Input(shape=(28, 28, 1)),
            layers.Conv2D(64, (3, 3), strides=(2, 2), padding="same"),
            layers.LeakyReLU(alpha=0.2),
            layers.Conv2D(128, (3, 3), strides=(2, 2), padding="same"),
            layers.LeakyReLU(alpha=0.2),
            layers.GlobalMaxPooling2D(),
            layers.Dense(1),
        ],
        name="discriminator",
    )
discriminator.summary()
```

### 自定义模型

以该GAN为例。继承了keras.Model

我们需要覆写的方法主要有两个：compile与train_step

```python
class GAN(keras.Model):
    loss_fn: object
    g_optimizer: keras.optimizers.Optimizer
    d_optimizer: keras.optimizers.Optimizer

    def call(self, inputs, training=None, mask=None):
        pass

    def get_config(self):
        pass

    def __init__(self, discriminator, generator, latent_dim):
        super(GAN, self).__init__()
        self.discriminator = discriminator
        self.generator = generator
        self.latent_dim = latent_dim

    def compile(self, d_optimizer, g_optimizer, loss_fn):
        super(GAN, self).compile()
        self.d_optimizer = d_optimizer
        self.g_optimizer = g_optimizer
        self.loss_fn = loss_fn

    def train_step(self, real_images):
        if isinstance(real_images, tuple):
            real_images = real_images[0]
        batch_size = tf.shape(real_images)[0]
        # 生成随机噪声
        random_latent_vectors = tf.random.normal(shape=(batch_size, self.latent_dim))
        generated_images = self.generator(random_latent_vectors)
        print("DEBUG")
        print(generated_images)

        combined_images = tf.concat([generated_images, real_images], axis=0)
        labels = tf.concat(
            [tf.ones((batch_size, 1)), tf.zeros((batch_size, 1))], axis=0
        )
        labels += 0.05 * tf.random.uniform(tf.shape(labels))

        with tf.GradientTape() as tape:
            predictions = self.discriminator(combined_images)
            d_loss = self.loss_fn(labels, predictions)
            grads = tape.gradient(d_loss, self.discriminator.trainable_weights)
            self.d_optimizer.apply_gradients(zip(grads, self.discriminator.trainable_weights))

        random_latent_vectors = tf.random.normal(shape=(batch_size, self.latent_dim))

        misleading_labels = tf.zeros((batch_size, 1))
        with tf.GradientTape() as tape:
            predictions = self.discriminator(self.generator(random_latent_vectors))
            g_loss = self.loss_fn(misleading_labels, predictions)
            grads = tape.gradient(g_loss, self.generator.trainable_weights)
            self.g_optimizer.apply_gradients(zip(grads, self.generator.trainable_weights))

        return {"d_loss": d_loss, "g_loss": g_loss}
    
    
```

#### compile方法

1. 传入优化器和损失函数
2. 调用父类的compile方法

传入的优化器与损失函数，可以在之后自己覆写的train_step方法中使用。

#### train_step方法

在编写train_step方法前，需要理清几个概念：

1. Model
2. Tensor
3. numpy

Model是指的是我们在之前使用keras定义的模型。模型的输入与输出都是Tensor。而不是具体的numpy的数据。

```python
random_latent_vectors = tf.random.normal(shape=(batch_size, self.latent_dim))
```

比如这个，作为一个tensor来说，实际每次运行带入的numpy数据都是不同的。尽管代码只有这么一行。

理清了以上概念后，流程与单独使用tensorflow时的流程相同：

1. 核心是用损失函数求loss。
2. 使用优化器优化loss。

注意的是

```python
with tf.GradientTape() as tape:
    predictions = self.discriminator(combined_images)
    d_loss = self.loss_fn(labels, predictions)
    grads = tape.gradient(d_loss, self.discriminator.trainable_weights)
    self.d_optimizer.apply_gradients(zip(grads, self.discriminator.trainable_weights))
```

之前的tensorflow中，直接使用optimizer的minimize(loss)基本上就可以了，但是在这里，需要先用gradient求梯度。

再使用apply_gradients去应用梯度

### 使用模型

这里需要有一个认知，Tensor自身**不仅仅是存储**了一种计算**图的结构**，也是可以**拥有数据**的。

```python
def use_model(latent_dim):
    batch_size = 3
    model_path = os.path.abspath(os.path.dirname(__file__)) + "\\model\\gan_keras_g"
    generator = keras.models.load_model(model_path)
    random_latent_vectors = tf.random.normal(shape=(batch_size, latent_dim))
    print(random_latent_vectors.numpy())
    generated_images = generator(random_latent_vectors)
    generated_images *= 255
    print(generated_images.numpy())
    for i in range(batch_size):
        img = keras.preprocessing.image.array_to_img(generated_images[i])
        img.save("./samples/generated_img_{i}.png".format(i=i))
```

---

```
[[-1.03255856e+00 -3.34520310e-01 -1.56069481e+00  4.74755615e-01
  -1.62119460e+00  1.72271848e+00  1.05746591e+00 -1.27776414e-01
   1.22075927e+00  2.07884967e-01  6.75365701e-02 -5.04939735e-01
   1.24207056e+00  7.31029809e-01  1.05800235e+00 -1.85781741e+00
  -2.48684749e-01  2.43300244e-01 -1.22526741e+00  1.48491681e-01
  -2.05933303e-01  1.53870869e+00  9.09546494e-01  1.36386037e+00
  -5.47508180e-01  2.80085862e-01  8.66873503e-01  1.32687235e+00
  -8.59568357e-01 -5.41154802e-01  2.64266551e-01  5.59059978e-01
  -1.34787455e-01 -1.89891040e+00 -3.32171887e-01  1.11424714e-01
  ...
   3.83331403e-02  4.61438060e-01  6.50607422e-02  1.71517062e+00
  -8.08648884e-01  1.55951369e+00  6.71431601e-01 -2.25786269e-01
  -2.66724753e+00 -7.31046557e-01  8.11338007e-01 -3.52870315e-01]
  ...
   1.84977785e-01  1.64972842e-02  3.41385640e-02  5.07674813e-01
   4.41104889e-01  5.98341823e-01 -9.33253527e-01 -2.26909548e-01]]
[[[[7.06594903e-03]
   [1.04763895e-08]
   [2.03260228e-10]
   ...

  [[1.54585291e-18]
   [1.93678823e-35]
   [0.00000000e+00]
   ...
   [0.00000000e+00]
   [0.00000000e+00]
   [1.43158277e-21]]
   ...
   [3.11048377e-20]
   [3.62676281e-17]
   [6.84130530e-09]]]]

```

可以看出，tensor内部其实是包含的有数据的，这点与tf1中使用placeHolder构建出来的计算图稍有不同。

### 灰度图的输出

```python
# generated_images 是tensor(包含数据)或者numpy对象似乎都可以
img = keras.preprocessing.image.array_to_img(generated_images[i])
# img = keras.preprocessing.image.array_to_img(generated_images[i].numpy())
img.save("./samples/generated_img_{i}.png".format(i=i))
```

