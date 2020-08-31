

# 卷积神经网络

## 常见操作

### 卷积

为了解决遇到的一系列计算机视觉的问题。

![image-20200721164031952](.\dp_10_1.png)

可以对图像进行模糊化，降低像素，这里就引入了卷积

![image-20200721164929563](.\dp_10_2.png)

![image-20200721165128348](.\dp_10_3.png)

![image-20200721165613318](.\dp_10_4.png)

![image-20200721165745220](.\dp_10_5.png)

![image-20200721170034796](.\dp_10_6.png)

![image-20200721170209716](.\dp_10_7.png)

![image-20200721221256020](.\dp_10_8.png)

![image-20200721221441673](.\dp_10_9.png)

### 池化

![image-20200721221623250](.\dp_10_10.png)

![image-20200721221720684](.\dp_10_11.png)



## 引入依赖

### urllib2

使用这个模块可以导入网络中的图片

```python
import urllib2
from PIL import Image

im_url = 'https://image.ibb.co/iFGs0x/kitty.png'
im = Image.open(urllib2.urlopen(im_url)).convert('L') # 读入一张灰度图的图片
```

## 图像问题

### 模块介绍

#### tf.nn.conv2d 卷积

![image-20200721223132363](.\dp_10_12.png)

##### input(tensor)

是一个4维tensor(batch ,h ,w ,channels)

batch 代表输入的图片个数

h代表输入的图片高度

w代表输入的图片宽度

channels代表输入图片的通道数（RGB图为3，灰度图为1）

##### filter(tensor)

卷积核，是一个4维tensor(k_h,k_w,in,out)

k_h 是卷积核的高度

k_w 是卷积核的宽度

in 是卷积需要作用的输入图片的通道数，简称**输入通道数**

out 是卷积核的个数，也称作**输出通道数**

##### strides(list)

移动步长(1,s_h,s_w,1)

s_h在高度方向的移动步长

s_w在宽度方向的移动步长

##### padding(string)

补零方法有'SAME'和'VALID'

'SAME'为自动补零。移动步长为1的时候，可以保证输出大小与输入大小相同。

'VALID'不会自动补零

##### 实例

```python
sobel_kernel = np.array([[-1,-1,-1],[-1,8,-1],[-1,-1,-1]],dtype=np.float32)
sobel_kernel = tf.constant(sobel_kernel,shape=(3,3,1,1))

# 补零方法
edge1 = tf.nn.conv2d(im,sobel_kernel,[1,1,1,1],'SAME',name='same_conv') 
# 补零方法
edge2 = tf.nn.conv2d(im,sobel_kernel,[1,1,1,1],'VALID',name='same_conv') 
```

#### tf.nn.max_pool 池化

![image-20200721224954993](.\dp_10_13.png)

##### value(tensor)

池化输入value (batch, h , w ,channels

与卷积输入的意义相同

##### ksize(list)

池化核的大小 ksize (batch,k_h,k_w,1)

意义与卷积核相同

##### strides(list)

移动步长 strides (1,s_h,s_w,1)

与卷积相同

##### padding(string)

补零方法，与卷积相同

#### np.squeeze 清除多余维度

在卷积和池化中，要显示图片都使用到了imshow

(1, 457, 388, 1) 转化为 (457, 388)

```python
plt.imshow(np.squeeze(edge1_np), cmap='gray')
plt.imshow(np.squeeze(pool2_np), cmap='gray')
```

squeeze的效果如下

```python
print(pool2_np.shape)
print(np.squeeze(pool2_np).shape)
# (1, 457, 388, 1)
# (457, 388)
```



### Coding

#### 1`引入依赖

```python
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt
import tensorflow as tf
```

#### 2`打开图片

```python
im = Image.open('D:\\Storage\\pycharmProjects\\demo\\cnn\\a.jpg').convert('L')
# im = Image.open('D:\\Storage\\pycharmProjects\\demo\\cnn\\a.jpg')
# im = Image.open(ulb.urlopen(im_url)).convert('L')
# 这里类型指定为浮点类型很重要
im = np.array(im, dtype='float32')
# 查看图片的shape
print(im.shape)
# 图片的数据类型因为之前被指定为了浮点类型，所以这里要使用astype，暂时转化为整型
plt.imshow(im.astype('uint8'), cmap='gray')
# plt.imshow(im.astype('uint8'))
plt.show()
```

补充一下im.shape是什么

```
如果用彩色通道打开，那么im.shape有三个dim
(915, 777, 3)
反之，如果关闭彩色通道，im.shape则仅有两个dim
(915, 777)
```

![image-20200723085745325](.\dp_10_14.png)

#### 3`卷积

```python
# 卷积使用的图片需要进行形状的预处理
im = tf.constant(im.reshape((1, im.shape[0], im.shape[1], 1)), name='input')
# 定义卷积核
sobel_kernel = np.array([[-1, -1, -1], [-1, 8, -1], [-1, -1, -1]], dtype=np.float32)
sobel_kernel = tf.constant(sobel_kernel, shape=(3, 3, 1, 1))

# 进行卷积
edge1 = tf.nn.conv2d(im, sobel_kernel, [1, 1, 1, 1], 'SAME', name='same_conv')
edge2 = tf.nn.conv2d(im, sobel_kernel, [1, 1, 1, 1], 'VALID', name='valid_conv')
```

#### 4`查看卷积结果

```python
sess = tf.Session()
edge1_np, edge2_np = sess.run([edge1, edge2])
print(edge1_np.shape)
print(edge2_np.shape)

plt.imshow(np.squeeze(edge1_np), cmap='gray')
plt.title("edge_same")
plt.show()

plt.imshow(np.squeeze(edge2_np), cmap='gray')
plt.title("edge_valid")
plt.show()
```

edge1与edge2的shape分别如下，可以看到'SAME'可以很好的保持原本图片的形状

```
(1, 915, 777, 1)
(1, 913, 775, 1)
```

实际上，valid与same的生成效果几乎差不多

![image-20200723090335822](.\dp_10_15.png)

#### 5`查看池化结果

```python
pool1 = tf.nn.max_pool(im, [1, 2, 2, 1], [1, 2, 2, 1], 'SAME', name='same_max_pool')
pool2 = tf.nn.max_pool(im, [1, 2, 2, 1], [1, 2, 2, 1], 'VALID', name='valid_max_pool')

pool1_np, pool2_np = sess.run([pool1, pool2])
print(pool1_np.shape)
print(pool2_np.shape)

plt.imshow(np.squeeze(edge1_np), cmap='gray')
plt.title("pool_same")
plt.show()

plt.imshow(np.squeeze(edge2_np), cmap='gray')
plt.title("pool_valid")
plt.show()
```

池化的大小会明显小于原图片（卷积）的大小，几乎缩小到原先的四分之一（长宽各二分之一）

```
(1, 458, 389, 1)
(1, 457, 388, 1)
```

池化的效果如下，保持原图像的信息，但是，可以明显降低计算量

![image-20200723091447780](.\dp_10_16.png)