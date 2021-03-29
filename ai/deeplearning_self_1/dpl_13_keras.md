# keras

keras，内部使用Tensorflow，整体提供高级API，从而快速搭建需要的神经网络。

## 目录

[TOC]



## 依赖、准备

### 0·依赖图

```
Keras                    2.4.3
pydot                    1.4.1
tensorflow-gpu           2.3.0
numpy                    1.18.5
matplotlib               2.2.5
```

### 1·keras导入

安装完keras之后，不必再导入tensorflow.keras。也为了后面能够方便的更改绘图的那个API

```
import keras
import keras.layers as layers
import tensorflow as tf
import os
import numpy as np
```

### 2·pydot绘图

请使用最新导入的pydot来覆盖掉原先的pydot

```
import pydot
from keras.utils.vis_utils import model_to_dot

keras.utils.vis_utils.pydot = pydot
```

再使用绝对路径进行绘图

```python
draw_path = os.path.abspath(os.path.dirname(__file__)) + "\\graph\\demo.png"
keras.utils.plot_model(model, draw_path)
```

## 基础使用方式

### 定义模型

#### 定义输入

这里定义输入的形状的时候

```python
inputs = keras.Input(shape=(784,))
print(inputs.shape)
print(inputs.dtype)
```

---

我们发现，这里定义的实际的维度，仍然是在列向量上。

```
(None, 784)
<dtype: 'float32'>
```

#### 定义中间层和输出

```python
x = layers.Dense(64, activation="relu")(inputs)
x = layers.Dense(64, activation="relu")(x)
outputs = layers.Dense(10)(x)
```

#### 完成模型定义

这里的模型，完成定义的时候，仅仅需要描述输入输出的网络关系即可，但是还没有指定损失函数的计算方式和优化器

```python
# 定义模型
model = keras.Model(inputs, outputs, name="mnist_model")
# 打印模型的形状
model.summary()
```

---

```
Model: "mnist_model"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
input_1 (InputLayer)         [(None, 784)]             0         
_________________________________________________________________
dense (Dense)                (None, 64)                50240     
_________________________________________________________________
dense_1 (Dense)              (None, 64)                4160      
_________________________________________________________________
dense_2 (Dense)              (None, 10)                650       
=================================================================
Total params: 55,050
Trainable params: 55,050
Non-trainable params: 0
_________________________________________________________________
```

#### keras编译模型(指定损失函数与优化器)

```python
model.compile(loss=keras.losses.SparseCategoricalCrossentropy(from_logits=True), optimizer=keras.optimizers.RMSprop(),
              metrics=["accuracy"])
```


### 获取数据集

#### keras获取mnist数据集

```python
(x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()
x_train = x_train.reshape(60000, 784).astype("float32") / 255
x_test = x_test.reshape(10000, 784).astype("float32") / 255
```


#### keras训练模型

```python
history = model.fit(x_train, y_train, batch_size=64, epochs=2, validation_split=0.2)
```

---

```
Epoch 1/2
750/750 [==============================] - 1s 1ms/step - loss: 0.3516 - accuracy: 0.8995 - val_loss: 0.1927 - val_accuracy: 0.9427
Epoch 2/2
750/750 [==============================] - 1s 897us/step - loss: 0.1664 - accuracy: 0.9508 - val_loss: 0.1461 - val_accuracy: 0.9571
```

#### keras测试模型

```python
test_scores = model.evaluate(x_test, y_test, verbose=2)
print("Test Loss ", test_scores[0])
print("Test Accuracy ", test_scores[1])
```

---

```
313/313 - 0s - loss: 0.1419 - accuracy: 0.9579
Test Loss  0.1418544501066208
Test Accuracy  0.9578999876976013
```

### 保存模型

```python
# 保存模型路径
modelPath = os.path.abspath(os.path.dirname(__file__)) + "\\model\\demo"
# 根据路径保存模型
model.save(modelPath)
# 打印模型相关信息
print(model.to_json())
del model
```

### 读取模型

```python
model = keras.models.load_model(modelPath)

test_scores = model.evaluate(x_test, y_test)
print("Test Loss ", test_scores[0])
print("Test Accuracy ", test_scores[1])
```

---

读取模型重新计算的accuracy为什么这么低？

```
Test Loss  0.14760807156562805
Test Accuracy  0.10199999809265137
```

之前的与这个相比，loss相同（我相信不是巧合），但是准确率的计算发生了什么?

```
Test Loss  0.14760807156562805
Test Accuracy  0.9575999975204468
```

后来，我发现，当你需要**重新衡量准确率**的时候，需要对原有模型进行**重新编译**。

```python
model = keras.models.load_model(modelPath)

model.compile(loss=keras.losses.SparseCategoricalCrossentropy(from_logits=True), optimizer=keras.optimizers.RMSprop(),
              metrics=["accuracy"])

test_scores = model.evaluate(x_test, y_test)
print("Test Loss ", test_scores[0])
print("Test Accuracy ", test_scores[1])
```

## 更多使用方式

### 卷积与池化

#### 卷积、池化的模型定义

```python
encoder_input = keras.Input(shape=(28, 28, 1), name="img")
x = layers.Conv2D(16, 3, activation="relu")(encoder_input)
x = layers.Conv2D(32, 3, activation="relu")(x)
x = layers.MaxPooling2D(3)(x)
x = layers.Conv2D(32, 3, activation="relu")(x)
x = layers.Conv2D(16, 3, activation="relu")(x)
encoder_output = layers.GlobalMaxPool2D()(x)

encoder = keras.Model(encoder_input, encoder_output, name="encoder")
encoder.summary()
```

---

MaxPooling2D让整体的维度有明显的下降，MaxPooling2D(3)中的这个参数3，代表池化核的大小为(3x3)，同时移动的步长也为(3,3)这样该池化核每次池化的面积都是不重复的。

GlobalMaxPool2D是覆盖全部元素一层的一次池化（可以看到4x4的一个通道看作一层，(4x4) x 16 ，这16层每层都做了一次全局池化）

```
Model: "encoder"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
img (InputLayer)             [(None, 28, 28, 1)]       0         
_________________________________________________________________
conv2d (Conv2D)              (None, 26, 26, 16)        160       
_________________________________________________________________
conv2d_1 (Conv2D)            (None, 24, 24, 32)        4640      
_________________________________________________________________
max_pooling2d (MaxPooling2D) (None, 8, 8, 32)          0         
_________________________________________________________________
conv2d_2 (Conv2D)            (None, 6, 6, 32)          9248      
_________________________________________________________________
conv2d_3 (Conv2D)            (None, 4, 4, 16)          4335      
_________________________________________________________________
global_max_pooling2d (Global (None, 16)                0         
=================================================================
```

#### 反卷积、反池化

简单对反卷积做一个介绍

![image-20200804111233203](.\imgs_13\image-20200804111233203.png)

反卷积的工作是:
$$
(2x2)\frac{use}{kernel}--> (4x4)
$$
原式通过矩阵相乘获得了(2x2)的输出（抛开维度信息是4个元素），观察式子，我们发现，为了能够获取到(4x4)的大小，我们让y = Cx再左乘一个C的转置，即为
$$
x^{'}=C^TCx=C^Ty
$$
维度信息 (16 x 4)(4 x 16)(16 x 1) -> (16 x 1)。最后将这个16进行reshape即可得到(4x4)的矩阵

通过运算可知，恢复得到的x与原先的x并不相同。

---

反池化相较于反卷积，就好理解的多了

![image-20200804112605028](.\imgs_13\image-20200804112605028.png)

---

紧接之前卷积、池化的代码，现在是反卷积与反池化的代码

```python
x = layers.Reshape((4, 4, 1))(encoder_output)
x = layers.Conv2DTranspose(16, 3, activation="relu")(x)
x = layers.Conv2DTranspose(32, 3, activation="relu")(x)
x = layers.UpSampling2D(3)(x)
x = layers.Conv2DTranspose(16, 3, activation="relu")(x)
decoder_output = layers.Conv2DTranspose(1, 3, activation="relu")(x)

auto_encoder = keras.Model(encoder_input, decoder_output, name="auto_encoder")
auto_encoder.summary()
```

---

查看打印信息，了解整体的网络结构

```
Model: "auto_encoder"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
img (InputLayer)             [(None, 28, 28, 1)]       0         
_________________________________________________________________
conv2d (Conv2D)              (None, 26, 26, 16)        160       
_________________________________________________________________
conv2d_1 (Conv2D)            (None, 24, 24, 32)        4640      
_________________________________________________________________
max_pooling2d (MaxPooling2D) (None, 8, 8, 32)          0         
_________________________________________________________________
conv2d_2 (Conv2D)            (None, 6, 6, 32)          9248      
_________________________________________________________________
conv2d_3 (Conv2D)            (None, 4, 4, 16)          4624      
_________________________________________________________________
global_max_pooling2d (Global (None, 16)                0         
_________________________________________________________________
reshape (Reshape)            (None, 4, 4, 1)           0         
_________________________________________________________________
conv2d_transpose (Conv2DTran (None, 6, 6, 16)          160       
_________________________________________________________________
conv2d_transpose_1 (Conv2DTr (None, 8, 8, 32)          4640      
_________________________________________________________________
up_sampling2d (UpSampling2D) (None, 24, 24, 32)        0         
_________________________________________________________________
conv2d_transpose_2 (Conv2DTr (None, 26, 26, 16)        4624      
_________________________________________________________________
conv2d_transpose_3 (Conv2DTr (None, 28, 28, 1)         145       
=================================================================
Total params: 28,241
Trainable params: 28,241
Non-trainable params: 0
_________________________________________________________________
```

### 使用函数多次定义组合的网络

注意，使用Model定义出来的model，model本身是可以传参的 y = model(x)

x与y都是layers。

```python
import tensorflow.keras as keras
import tensorflow.keras.layers as layers
import os


def get_model():
    inputs = keras.Input(shape=(128,))
    outputs = layers.Dense(1)(inputs)
    return keras.Model(inputs, outputs)


def main():
    model1 = get_model()
    model2 = get_model()
    model3 = get_model()

    inputs = keras.Input(shape=(128,))
    y1 = model1(inputs)
    y2 = model2(inputs)
    y3 = model3(inputs)
    outputs = layers.average([y1, y2, y3])
    ensemble_model = keras.Model(inputs=inputs, outputs=outputs, name="function_define_model")
    ensemble_model.summary()


main()
```

---

```
Model: "function_define_model"
__________________________________________________________________________________________________
Layer (type)                    Output Shape         Param #     Connected to                     
==================================================================================================
input_4 (InputLayer)            [(None, 128)]        0                                            
__________________________________________________________________________________________________
functional_1 (Functional)       (None, 1)            129         input_4[0][0]                    
__________________________________________________________________________________________________
functional_3 (Functional)       (None, 1)            129         input_4[0][0]                    
__________________________________________________________________________________________________
functional_5 (Functional)       (None, 1)            129         input_4[0][0]                    
__________________________________________________________________________________________________
average (Average)               (None, 1)            0           functional_1[0][0]               
                                                                 functional_3[0][0]               
                                                                 functional_5[0][0]               
==================================================================================================
Total params: 387
Trainable params: 387
Non-trainable params: 0
__________________________________________________________________________________________________
```

### 具有多个输入输出的网络

多个输入和输出时：

compile的损失函数需要针对每一个输出进行单独制定，如果可以，还可以为每一个损失函数指定响应的权重值

fit进行训练时，需要指定输入与输出的name。

```python
class SeveralModel:
    _model = None

    def __init__(self):
        title_input = keras.Input(shape=(None,), name="title")
        body_input = keras.Input(shape=(None,), name="body")
        tags_input = keras.Input(shape=(num_tags,), name="tags")

        title_features = layers.Embedding(num_words, 64, name="title_emb")(title_input)
        body_features = layers.Embedding(num_words, 64, name="body_emb")(body_input)

        title_features = layers.LSTM(128)(title_features)
        body_features = layers.LSTM(32)(body_features)

        x = layers.concatenate([title_features, body_features, tags_input])

        priority_pred = layers.Dense(1, name="priority")(x)
        department_pred = layers.Dense(num_departments, name="department")(x)

        model = keras.Model(inputs=[title_input, body_input, tags_input], outputs=[priority_pred, department_pred])

        self._model = model

    def print(self):
        self._model.summary()
        draw_path = os.path.abspath(os.path.dirname(__file__)) + "\\structure\\several_demo_model.png"
        keras.utils.plot_model(self._model, draw_path, show_shapes=True)

    def compile(self):
        self._model.compile(optimizer=keras.optimizers.RMSprop(1e-3), loss=[
            keras.losses.BinaryCrossentropy(from_logits=True),
            keras.losses.CategoricalCrossentropy(from_logits=True)
        ], loss_weights=[1.0, 0.2])

    def train(self, title_data, body_data, tags_data, priority_targets, dept_targets):
        self._model.fit({
            'title': title_data, "body": body_data, "tags": tags_data
        }, {
            "priority": priority_targets, "department": dept_targets
        }, epochs=2, batch_size=32)
```

#### ResNet

```python
class ResNetModel:
    _model = None

    def __init__(self):
        inputs = keras.Input(shape=(32, 32, 3), name="img")
        x = layers.Conv2D(32, 3, activation="relu")(inputs)
        x = layers.Conv2D(64, 3, activation="relu")(x)
        block_1_output = layers.MaxPooling2D(3)(x)

        x = layers.Conv2D(64, 3, activation="relu", padding="same")(block_1_output)
        x = layers.Conv2D(64, 3, activation="relu", padding="same")(x)
        block_2_output = layers.add([x, block_1_output])

        x = layers.Conv2D(64, 3, activation="relu", padding="same")(block_2_output)
        x = layers.Conv2D(64, 3, activation="relu", padding="same")(x)
        block_3_output = layers.add([x, block_2_output])

        x = layers.Conv2D(64, 3, activation="relu")(block_3_output)
        x = layers.GlobalAveragePooling2D()(x)
        x = layers.Dense(256, activation="relu")(x)
        x = layers.Dropout(0.5)(x)
        outputs = layers.Dense(10)(x)

        self._model = keras.Model(inputs, outputs, name="toy_resnet")

    def print(self):
        draw_path = os.path.abspath(os.path.dirname(__file__)) + "\\structure\\resnet.png"
        keras.utils.plot_model(self._model, draw_path, show_shapes=True)

    def compile(self):
        self._model.compile(optimizer=keras.optimizers.RMSprop(1e-3),
                            loss=keras.losses.CategoricalCrossentropy(from_logits=True),
                            metrics=["acc"])

    def train(self, x_train, y_train):
        self._model.fit(x_train, y_train, batch_size=64, epochs=1, validation_split=0.2)


def create_data():
    (x_train, y_train), (x_test, y_test) = keras.datasets.cifar10.load_data()
    x_train = x_train.astype("float32") / 255.0
    x_test = x_test.astype("float32") / 255.0
    y_train = keras.utils.to_categorical(y_train, 10)
    y_test = keras.utils.to_categorical(y_test, 10)
    return x_train, y_train, x_test, y_test


def main():
    model = ResNetModel()
    model.print()
    model.compile()
    datas = create_data()
    model.train(datas[0][:2000], datas[1][:2000])
```

---

```
25/25 [==============================] - 2s 88ms/step - loss: 2.3038 - acc: 0.1112 - val_loss: 2.2743 - val_acc: 0.1075
```

### 自定义层

从自定义层里也看出，输入与输出，都对应的是tensorflow中的tensor

```python
class CustomDense(layers.Layer):
    def __init__(self, units=32):
        super(CustomDense, self).__init__()
        self.units = units
        self.w = None
        self.b = None

    def build(self, input_shape):
        self.w = self.add_weight(
            shape=(input_shape[-1], self.units),
            initializer="random_normal",
            trainable=True,
        )
        self.b = self.add_weight(
            shape=(self.units,), initializer="random_normal", trainable=True
        )

    def call(self, inputs):
        return tf.matmul(inputs, self.w) + self.b

    def get_config(self):
        return {"units": self.units}


def main():
    inputs = keras.Input((4,))
    outputs = CustomDense(10)(inputs)
    model = keras.Model(inputs, outputs)

    draw_path = os.path.abspath(os.path.dirname(__file__)) + "\\structure\\custom_dense.png"
    keras.utils.plot_model(model, draw_path, show_shapes=True)
    model.summary()


if __name__ == '__main__':
    main()
```

---

```
Model: "functional_1"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
input_1 (InputLayer)         [(None, 4)]               0         
_________________________________________________________________
custom_dense (CustomDense)   (None, 10)                50        
=================================================================
Total params: 50
Trainable params: 50
Non-trainable params: 0
_________________________________________________________________
```

### 自定义模型（tf2与tf1的不同处）

在这段自定义模型的代码中，输入并不是之前的None型的占位符，我们传入一段确切的数值也是可以作为输入的

**tf.zeros((1, 32))**，所起的作用就相当于**tf1中的占位符**。其中32代表**输入的维度**，1则是一个**输入样本最小的个数**。

在顺序模型中，还会对此做详细解释。

```python
import keras
import keras.layers as layers
import tensorflow as tf
import os
import numpy as np

import pydot
from keras.utils.vis_utils import model_to_dot

keras.utils.vis_utils.pydot = pydot


class MLP(keras.Model):

    def get_config(self):
        pass

    def __init__(self, **kwargs):
        super(MLP, self).__init__(**kwargs)
        self.dense_1 = layers.Dense(64, activation='relu')
        self.dense_2 = layers.Dense(10)

    def call(self, inputs):
        x = self.dense_1(inputs)
        return self.dense_2(x)

    def print(self):
        self.summary()
        draw_path = os.path.abspath(os.path.dirname(__file__)) + "\\structure\\custom_model.png"
        keras.utils.plot_model(self, draw_path)


def main():
    # Instantiate the model.
    mlp = MLP()
    # _ = mlp(tf.zeros((1, 32))) # tf2中，不在保有占位符这种东西
    _ = mlp(keras.Input(shape=(784,)))
    mlp.print()


if __name__ == '__main__':
    main()

```

---

```
Model: "mlp"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
dense (Dense)                (None, 64)                50240     
_________________________________________________________________
dense_1 (Dense)              (None, 10)                650       
=================================================================
Total params: 50,890
Trainable params: 50,890
Non-trainable params: 0
_________________________________________________________________
```

### 顺序模型

#### 什么是顺序模型

```
model = keras.Sequential(
    [
        layers.Dense(2, activation="relu"),
        layers.Dense(3, activation="relu"),
        layers.Dense(4),
    ]
)
```

---

```
model = keras.Sequential()
model.add(layers.Dense(2, activation="relu"))
model.add(layers.Dense(3, activation="relu"))
model.add(layers.Dense(4))
```

相较于Model定义模型，Sequential定义模型具有更高的可变度

同样顺序模型定义的时候，没有使用keras.Input指明输入向量的维度，因此，第一层的模型的维度，总是在接受到第一个输入之后才能确定的。

```python
model = keras.Sequential(
    [
        layers.Dense(2, activation="relu"),
        layers.Dense(3, activation="relu"),
        layers.Dense(4),
    ]
)
# 在这个之后 y = model(x) 之后，才能确定输入模型的维度
x = tf.ones((1, 4))
y = model(x)
```

#### 常见的使用场景

```python
model = keras.Sequential()
model.add(keras.Input(shape=(250, 250, 3)))  # 250x250 RGB images
model.add(layers.Conv2D(32, 5, strides=2, activation="relu"))
model.add(layers.Conv2D(32, 3, activation="relu"))
model.add(layers.MaxPooling2D(3))

# Can you guess what the current output shape is at this point? Probably not.
# Let's just print it:
model.summary()

# The answer was: (40, 40, 32), so we can keep downsampling...

model.add(layers.Conv2D(32, 3, activation="relu"))
model.add(layers.Conv2D(32, 3, activation="relu"))
model.add(layers.MaxPooling2D(3))
model.add(layers.Conv2D(32, 3, activation="relu"))
model.add(layers.Conv2D(32, 3, activation="relu"))
model.add(layers.MaxPooling2D(2))

# And now?
model.summary()

# Now that we have 4x4 feature maps, time to apply global max pooling.
model.add(layers.GlobalMaxPooling2D())

# Finally, we add a classification layer.
model.add(layers.Dense(10))
```

---

```
Model: "sequential_6"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
conv2d (Conv2D)              (None, 123, 123, 32)      2432      
_________________________________________________________________
conv2d_1 (Conv2D)            (None, 121, 121, 32)      9248      
_________________________________________________________________
max_pooling2d (MaxPooling2D) (None, 40, 40, 32)        0         
=================================================================
Total params: 11,680
Trainable params: 11,680
Non-trainable params: 0
_________________________________________________________________
Model: "sequential_6"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
conv2d (Conv2D)              (None, 123, 123, 32)      2432      
_________________________________________________________________
conv2d_1 (Conv2D)            (None, 121, 121, 32)      9248      
_________________________________________________________________
max_pooling2d (MaxPooling2D) (None, 40, 40, 32)        0         
_________________________________________________________________
conv2d_2 (Conv2D)            (None, 38, 38, 32)        9248      
_________________________________________________________________
conv2d_3 (Conv2D)            (None, 36, 36, 32)        9248      
_________________________________________________________________
max_pooling2d_1 (MaxPooling2 (None, 12, 12, 32)        0         
_________________________________________________________________
conv2d_4 (Conv2D)            (None, 10, 10, 32)        9248      
_________________________________________________________________
conv2d_5 (Conv2D)            (None, 8, 8, 32)          9248      
_________________________________________________________________
max_pooling2d_2 (MaxPooling2 (None, 4, 4, 32)          0         
=================================================================
Total params: 48,672
Trainable params: 48,672
Non-trainable params: 0
_________________________________________________________________
```