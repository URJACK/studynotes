# AlexNet

## 导入数据集(下载到本机的数据集)

```python
import pickle
import numpy as np
from keras import utils as kutils


def unpickle(file):
    with open(file, 'rb') as fo:
        obj = pickle.load(fo, encoding='bytes')
    return obj


def main():
    obj = unpickle("D:\\Storage\\datasets\\cifar-10-batches-py\\data_batch_1")
    print(obj)
    labels = obj[b'labels']  # 获取到b'obj'对象
    datas = obj[b'data']  # 获取到b'data'对象
    print(type(labels))
    labels = np.array(labels)  # 将 labels 转为 ndarray 类型
    print(type(labels))
    print("===========")
    print(type(datas))
    print("===========")
    print(labels.shape)  # 查看labels 与 datas 的形状
    print(datas.shape)
    print("===========")
    print(labels)
    labels = kutils.to_categorical(labels)  # 将labels 进行 one-hot 编码
    print(labels)


if __name__ == '__main__':
    main()

```

---

```
{b'batch_label': b'training batch 1 of 5', b'labels': [6, 9, 9....... 1, 1, 5], b'data': array([[ 59,  43,  50, ..., 140,  84,  72],
       ...,
       [ 62,  61,  60, ..., 130, 130, 131]], dtype=uint8), b'filenames': [b'leptodactylus_pentadactylus_s_000004.png', ....... b'car_s_002296.png', b'estate_car_s_001433.png', b'cur_s_000170.png']}
<class 'list'>
<class 'numpy.ndarray'>
===========
<class 'numpy.ndarray'>
===========
(10000,)
(10000, 3072)
===========
[6 9 9 ... 1 1 5]
[[0. 0. 0. ... 0. 0. 0.]
 [0. 0. 0. ... 0. 0. 1.]
 [0. 0. 0. ... 0. 0. 1.]
 ...
 [0. 1. 0. ... 0. 0. 0.]
 [0. 1. 0. ... 0. 0. 0.]
 [0. 0. 0. ... 0. 0. 0.]]
```

## 创建数据集获取类

### 获取数据集路径

![image-20200818082913604](.\imgs_15\image-20200818082913604.png)

因为数据集在本机中按如下规律存储，所以我需要有一个可以获取到他们路径的办法

在代码中，我使用了**glob**.glob(folder_pattern_jpg) 去获取到所有单个数据集文件的路径"python-string类型"

为了打乱每一个数据集文件的训练顺序，我使用了np.random.shuffle()函数

实际解析数据集的过程，使用**pickle**这个包，因为原数据集是用python对象序列化得到的，针对不同的数据集，解析的过程也有所不同。

```python
dataGenerator = DataGenerator('D:\\Storage\\datasets\\cifar-10-batches-py\\*.data', 64)
```

```python
class DataGenerator:
    # 此处要求所有的单个 "数据集文件的长度" 都是 "一样" 的
    def __init__(self, folder_pattern_jpg, batch_size):
        self.batch_size = batch_size
        data_names = glob.glob(folder_pattern_jpg)  # 读取数据集文件名称
        np.random.shuffle(data_names)  # 打乱数据集文件名称的顺序
        self.data_names = data_names  # 将获取到的数据集文件名称中放入成员变量，方便后续的迭代器调用
        obj = unpickle(data_names[0])  # （关键）从单个文件中，解析出数据
        self.per_set_len = len(obj[b'labels'])  # 提取出labels，因为labels是python-list类型，可以直接用len获取到样本个数
        self.sets_num = len(self.data_names)  # 获取到数据集一共有sets_num个
        self.all_sets_len = self.per_set_len * self.sets_num  # 得到总的数据集个数
        # 规划训练的总步数
        self.steps = self.per_set_len // batch_size
        # 如果没有整除，需要额外加上一步
        if self.per_set_len % batch_size != 0:
            self.steps = self.steps + 1
        # 单个数据集需要走的步数，变为所有数据集需要的步数
        self.steps = self.steps * self.sets_num
```

### 数据的预处理

这里边，对x的预处理也十分重要，因为在cifar-10中数据集的格式为 （n x 3072）

其中3072的组成是  1024个R，1024个G，1024个B。即 (n x 3 x 32 x 32)

但是keras中，卷积网络需要的结构必须是(n x 32 x 32 x 3)

为了将高维度与低维度进行调换，我使用了一个脑残操作，最终连图片都输出不正确，下面看看正确的输出方式吧

详见：x_preprocess(x)

```python
def __len__(self):
    return self.steps

@staticmethod
def x_preprocess(x):
    # x = np.reshape(x, (-1, 3, 1024))
    # x = x.T  # 1024 x 3
    # x = np.reshape(x, (-1, 32, 32, 3))

    # 非常重要，必须采用下面这种预处理的办法
    x = np.reshape(x, (-1, 3, 32, 32))
    x = x.transpose(0, 2, 3, 1)
    return x / 255 * 2 - 1
   
@staticmethod
def y_preprocess(y):
    return kutils.to_categorical(y)
```

覆写len（其实也可以单独另起一个新的函数），返回所有数据集训练完毕后的次数

y_preprocess 是对y做one-hot编码。重要性不再赘述。

transpose的作用图像化后大概如下

![image-20200818091918850](.\imgs_15\image-20200818091918850.png)

```python
def __iter__(self):
    while True:
        for (i, f) in enumerate(self.data_names):
            obj = unpickle(f)
            labels = obj[b'labels']  # 获取到b'obj'对象
            datas = obj[b'data']  # 获取到b'data'对象
            labels = np.array(labels)  # 将 labels 转为 ndarray 类型
            datas = self.x_preprocess(datas)  # 对datas 进行归一化
            labels = self.y_preprocess(labels)  # 将labels 进行 one-hot 编码
            start = 0
            end = self.batch_size
            while end <= self.per_set_len:
                X = datas[start:end]
                Y = labels[start:end]
                yield X, Y
                start = start + self.batch_size
                end = end + self.batch_size
            # 调整越界的end
            if start < self.per_set_len:
                yield datas[start:self.per_set_len], labels[start:self.per_set_len]
```

迭代器方法中，我们一定要每次给数据集yield出去 batch_size个数据，最重要的一个原因当然是我们的步数是通过batch_size计算得到的

这里注意一下对越界情况的处理（即，一次如果不满batch_size个）。

## 模型的创建

```python
def createAlexNet(channel: int, height: int, width: int) -> keras.Model:
    inputs = keras.Input(shape=(height, width, channel))
    x = keras.layers.Conv2D(64, (5, 5), strides=(1, 1), padding="valid",
                            kernel_initializer=keras.initializers.RandomNormal(0, 0.02))(inputs)
    x = keras.layers.MaxPooling2D(pool_size=(3, 3), strides=(2, 2), padding="valid")(x)
    x = keras.layers.BatchNormalization()(x)
    x = keras.layers.Conv2D(64, (5, 5), strides=(1, 1), padding="valid",
                            kernel_initializer=keras.initializers.RandomNormal(0, 0.02))(x)
    x = keras.layers.MaxPooling2D(pool_size=(3, 3), strides=(2, 2), padding="valid")(x)
    x = keras.layers.BatchNormalization()(x)
    x = keras.layers.Flatten()(x)
    x = keras.layers.Dense(4096, kernel_initializer=keras.initializers.RandomNormal(0, 0.02))(x)
    x = keras.layers.BatchNormalization()(x)
    x = keras.layers.Dense(384, kernel_initializer=keras.initializers.RandomNormal(0, 0.02))(x)
    x = keras.layers.BatchNormalization()(x)
    x = keras.layers.Dense(192, kernel_initializer=keras.initializers.RandomNormal(0, 0.02))(x)
    x = keras.layers.BatchNormalization()(x)
    outputs = keras.layers.Dense(10, kernel_initializer=keras.initializers.RandomNormal(0, 0.02))(x)
    model = keras.Model(inputs, outputs)
    return model


def compileAlexNet(alexnet: keras.Model):
    alexnet.compile(optimizer=keras.optimizers.RMSprop(1e-4, 0.99),
                    loss=keras.losses.CategoricalCrossentropy(from_logits=True),
                    metrics=["acc"])
```

##  训练回调

```python
class Trainner(keras.callbacks.Callback):
    alexNet: keras.Model
    losses: list
    accs: list

    def __init__(self, alexNet):
        print("Trainner Init")
        self.alexNet = alexNet
        self.losses = []
        self.accs = []

    def on_epoch_end(self, epoch, logs=None):
        print("Epoch END")
        self.alexNet.save_weights('./model/alex_net.weights')
        loss = logs.get('loss')
        acc = logs.get('acc')
        self.losses.append(loss)
        self.accs.append(acc)

    def on_train_end(self, logs=None):
        losses = np.array(self.losses)
        accs = np.array(self.accs)
        style.use('ggplot')
        plt.plot(losses, color='b', label='line one', linewidth=3)
        plt.plot(accs, color='r', label='line two', linewidth=5)
        plt.title('losses and accuracy')
        plt.ylabel('Y axis')
        plt.xlabel('X axis')
        plt.legend()  # 会出现每根线的label 在图的右上角
        plt.grid(True, color='k')
        plt.show()
```

## 开始训练

```python
def train():
    dataGenerator = DataGenerator('D:\\Storage\\datasets\\cifar-10-batches-py\\*.data', 64)
    alexNet: keras.Model = createAlexNet(3, 32, 32)
    alexNet.summary()
    compileAlexNet(alexNet)

    trainner = Trainner(alexNet)
    alexNet.load_weights('./model/alex_net.weights')
    alexNet.fit_generator(dataGenerator.__iter__(), steps_per_epoch=len(dataGenerator), epochs=8, callbacks=[trainner])
```

## 测试训练效果

```python
def mapping(index):
    arr = ['飞机', '汽车', '鸟', '猫', '鹿', '狗', '青蛙', '马', '船', '卡车']
    return arr[index]
```

---

```python
def use():
    dim = 32
    height = 8
    width = 8
    alexNet: keras.Model = createAlexNet(3, 32, 32)
    alexNet.load_weights('./model/alex_net.weights')
    dataGenerator = DataGenerator('D:\\Storage\\datasets\\cifar-10-batches-py\\*.data', height * width)
    for (datas, labels) in dataGenerator:
        figure = np.zeros((height * dim, width * dim, 3))
        results: np.ndarray = []
        ans_list: list[str] = []
        for i in range(height):
            for j in range(width):
                data = datas[i * width + j]
                figure[i * dim:(i + 1) * dim, j * dim:(j + 1) * dim] = (data + 1) / 2 * 255
                value = alexNet.predict(data.reshape(1, dim, dim, 3))
                value = np.argmax(value, axis=1)[0]
                value = mapping(value)
                ans_list.append(value)
        index: int = 0
        printValue: str = ""
        for value in ans_list:
            index = index + 1
            printValue = printValue + value + "\t"
            if index == width:
                print(printValue)
                printValue = ""
                index = 0
        imageio.imwrite('samples/a.png', figure)
        break
```