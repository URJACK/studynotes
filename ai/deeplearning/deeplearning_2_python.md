# Python

## 常见用法

### 列表

```python
>>> x = [3,1,2]
>>> print(x,x[2])
[3, 1, 2] 2
```

```python
>>> x = [3,1,2]
>>> x.append('bar')
>>> print(x)
[3, 1, 2, 'bar']
```

### 集合

```python
animals = {'cat', 'dog'}
print('cat' in animals)   # 验证该元素是否在集合中
print('fish' in animals)
animals.add('fish')       # 在集合里面添加元素
print('fish' in animals)
print(len(animals))       # 打印出集合的元素个数
animals.add('cat')        
print(len(animals))
animals.remove('cat')     # 从集合里面移除某个元素
print(len(animals))
```

```
True
False
True
3
3
2
```

### 字典

```python
>>> dic = {'cat':'cute','dog':'cool'}
>>> print(dic['cat'])
cute
```

### 元组

```python
d = {(x, x+1): x for x in range(10)} # 使用元组创造一个字典
t = (5, 6)        # 创造一个元组
print(type(t))    # 打印出元组的类型
print(d[t])       # 打印出字典中对应于(5, 6)的元素
print(d[(1, 2)])  # 打印出字典中对应于(1, 2)的元素
```

### 循环

```python
>>> for animal in animals:
...     print(animal)
...
cat
dog
monkey
```

### 函数

```python
>>> def sign(x):
...     if x > 0:
...             return 'positive'
...     elif x < 0:
...             return 'negative'
...     else:
...             return 'zero'
...
>>> for x in [-1,0,1]:
...     print(sign(x))
...
negative
zero
positive
```

### 类

```python
>>> class Student(object):
...     def __init__(self,name,score):
...             self.name = name
...             self.score = score
...     def print_score(self):
...             print('{}:{}'.format(self.name,self.score))
...
>>> a = Student('fangzhou',99)
>>> b = Student('zhanglingfei',80)
>>> a.print_score()
fangzhou:99
>>> b.print_score()
zhanglingfei:80
```

### Print

占位符、打印需要的位数

```
print('rand_uniform: %.4f' % sess.run(rand_uniform))
```



## numpy

### 数组

```
>>> a = np.array([])
>>> a = np.array([1,2,3])
>>> a
array([1, 2, 3])
>>> print(a.shape)
(3,)
>>> b = np.array([[1,2,3],[4,5,6]])
>>> b
array([[1, 2, 3],
       [4, 5, 6]])
>>> print(b.shape)
(2, 3)
```

### 数组索引

(左边是闭区间，右边是开区间)

```
>>> a = np.array([[1,2,3,4],[5,6,7,8],[9,10,11,12]])
>>> print(a[0,1])
2
>>> print(a[1,:])
[5 6 7 8]
>>> print(a[1,2:])
[7 8]
>>> print(a[1,:2])
[5 6]
```

### 科学计算

```
>>> a = np.array([[1,2],[3,4]])
>>> b = np.array([[5,6],[7,8]])
>>> a
array([[1, 2],
       [3, 4]])
>>> b
array([[5, 6],
       [7, 8]])
>>> a + b
array([[ 6,  8],
       [10, 12]])
>>> a * b		#对应元素作乘法
array([[ 5, 12],
       [21, 32]])
>>> a.dot(b) 	#线性代数中的矩阵乘法
array([[19, 22],
       [43, 50]])
```

### 高级功能

#### sum求和

```
>>> a
array([[1, 2],
       [3, 4]])
>>> b
array([[5, 6],
       [7, 8]])
>>> np.sum(a)		 #所有元素求和
10
>>> np.sum(b)
26
>>> np.sum(a,axis=0) #列求和
array([4, 6])
>>> np.sum(a,axis=1) #行求和
array([3, 7])
```

#### stack

这是一个让人有些疑惑的函数

```python
import numpy as np

x1 = np.arange(9).reshape((3, 3))
x2 = np.arange(10, 19, 1).reshape((3, 3))

print(x1.shape)
print(x2.shape)

y2 = np.stack((x1, x2), axis=0)

print(y2.shape)
print(y2[0].shape)  # x1
print(y2[1].shape)  # x2
print(y2)
```

```
(3, 3)
(3, 3)
(2, 3, 3)
(3, 3)
(3, 3)
[[[ 0  1  2]
  [ 3  4  5]
  [ 6  7  8]]

 [[10 11 12]
  [13 14 15]
  [16 17 18]]]
```

我们发现当axis=0的时候，整个流程是非常清晰的。只需要取y[0]就可以访问到x1，y[1]就可以访问到x2

```
y[0][a][b] = x1[a][b]
y[1][a][b] = x2[a][b]
```

那如果axis等于其他数值呢?

```python
y3 = np.stack((x1, x2), axis=1)

print(y3.shape)
print(y3)
```

```
(3, 2, 3)
[[[ 0  1  2]
  [10 11 12]]

 [[ 3  4  5]
  [13 14 15]]

 [[ 6  7  8]
  [16 17 18]]]
```

```
由上述条件可知，当y3[a][0][b] == x1[a][b]
```

```python
y4 = np.stack((x1, x2), axis=2)
print(y4.shape)
print(y4)
```

```
(3, 3, 2)
[[[ 0 10]
  [ 1 11]
  [ 2 12]]

 [[ 3 13]
  [ 4 14]
  [ 5 15]]

 [[ 6 16]
  [ 7 17]
  [ 8 18]]]
```

```python
同理可推 y4[a][b][0] = x1[a][b]
y4[a][b][1] = x2[a][b]
axis = 2 对于这个初始都是"二维"的x来说，已经是最大的axis了
```

#### squeeze

这个方法可以很快的清除掉数组的所有维度信息

```python
import numpy as np

print("x1--------")
x1 = np.arange(3).reshape((1, 3))
print(x1.shape)
print(x1)
print("x2--------")
x2 = np.squeeze(x1)
print(x2.shape)
print(x2)
print("x3--------")
x3 = np.arange(3).reshape((3, 1))
print(x3.shape)
print(x3)
print("x4--------")
x4 = np.squeeze(x3)
print(x4.shape)
print(x4)
```

```
x1--------
(1, 3)
[[0 1 2]]
x2--------
(3,)
[0 1 2]
x3--------
(3, 1)
[[0]
 [1]
 [2]]
x4--------
(3,)
[0 1 2]
```

squeeze之后元素，就向量化了

## Matplotlib

这是Python中的画图工具，可以画2D的图像，也可以画3D的图像

```
import numpy as np
import matplotlib.pyplot as plt

x = np.arange(0, 8, 0.1)
y = np.sin(x)

plt.plot(x, y)
plt.show()
```

## Pillow

pillow是Python中官方的图像处理库，有很多对图片的操作，使用pip install Pillow进行安装，更多的信息，可以通过官方文档查看。

```python
>>> import numpy as np
>>> import PIL.Image as IMG
>>> img = IMG.open("D:\\Documents\\a.png")
>>> print(np.array(img)) #打印图像数据
>>> img.show() #调用图片显示接口
>>> new_img = img.transpose(IMG.FLIP_LEFT_RIGHT) #从左往右进行一次反转得到新图片
>>> new_img.show()
>>> new_img.save("D:\\Documents\\b.png") #保存图片
```

