# TensorFlow

## 依赖

老师的课程中，使用的是TensorFlow 1，而我使用的是TensorFlow 2。所以有一些属性是用不了的，比如tf.Session()。

为了解决问题，再引用TensorFlow的时候，应该使用该段代码

```
import tensorflow.compat.v1 as tf
tf.disable_eager_execution()
```

eager 模式是在 TF 1.4 版本之后引入的，据相关报道：在 TF 2.0(预计 18 年年底发布) 之后将会把 eager 模式变为默认执行模式；



## 运行机制

### session

tensor：TensorFlow框架里的两个数

下面这个过程，即为“构造图”，initializer也属于构造图的过程

```
>>> a = tf.constant(32)
>>> b = tf.constant(10)
>>> c = tf.add(a,b)
>>> a
<tf.Tensor: id=0, shape=(), dtype=int32, numpy=32>
>>> b
<tf.Tensor: id=1, shape=(), dtype=int32, numpy=10>
>>> c
<tf.Tensor: id=2, shape=(), dtype=int32, numpy=42>
```

**必须**在“构造图”完成后，**才可以**“运行图”

```
>>> sess = tf.Session()
>>> sess.run(a)
32
>>> sess.run(c)
42
```

```
>>> py_a = sess.run(a)
>>> print(type(py_a))
<class 'numpy.int32'>
>>> print(py_a)
32
>>> py_r = sess.run([a,b,c])
>>> print(type(py_r))
<class 'list'>
>>> print(py_r)
[32, 10, 42]
```

关闭图

```
>>> sess.close()
```

### interactiveSession

构造图过程省略

运行图，开启session

```
 sess = tf.InteractiveSession()
```



## Tensor

### 常量

tensor的类型是多样的

```python
>>> a = tf.constant("hello")
>>> b = tf.constant(True)
>>> c = tf.constant([1,2],dtype=tf.int32)
>>> d = tf.constant([1,2],dtype=tf.float32)
```

```python
>>> sess.run(a)
b'hello'
>>> sess.run(b)
True
>>> sess.run(c)
array([1, 2])
>>> sess.run(d)
array([1., 2.], dtype=float32)
```

每一个tensor都可以有自己的名字，名字":0"代表属于第0块GPU

```python
>>> a = tf.constant("hello",name="a")
>>> a
<tf.Tensor 'a:0' shape=() dtype=string>
>>> b = tf.constant("hello",name="fangzhou")
>>> b
<tf.Tensor 'fangzhou:0' shape=() dtype=string>
>>> c = tf.constant("hello",name="fangzhou")
>>> c
<tf.Tensor 'fangzhou_1:0' shape=() dtype=string>
```

name有什么用呢？已经有变量名了呀。

name的作用在于，如果最终这个session被保存的时候：

​	如果一个变量有name，那么存储的时候，就以name存储

​	当然我觉得如果一个变量没有name，那么存储的名字就是一个Const_1，Const_2这种类型的

### 变量

定义变量

```python
>>> var_a = tf.Variable(0,dtype=tf.int32)
>>> var_b = tf.Variable([1,2],dtype=tf.int32)
>>> var_w = tf.Variable(tf.zeros((1024,10)))
>>> var_a
<tf.Variable 'Variable:0' shape=() dtype=int32>
>>> var_b
<tf.Variable 'Variable_2:0' shape=(2,) dtype=int32>
>>> var_w
<tf.Variable 'Variable_3:0' shape=(1024, 10) dtype=float32>
```

#### session变量的初始化

全局变量初始化

```python
>>> init = tf.global_variables_initializer()
>>> sess.run(init)
```

局部变量初始化（即便是一个变量，也必须是数组类型）

```
>>> init_ab = tf.variables_initializer([var_a,var_b])
>>> sess.run(init_ab)
```

单个变量初始化

```
>>> sess.run(var_w.initializer)
```

#### interactiveSession变量初始化

sess.run(xxxx) 改为 xxx.run() 即可

```python
# 构造图
init = tf.global_variables_initializer()
# 运行图
sess = tf.InteractiveSession()
init.run()

print(sess.run(a))
```

#### 变量赋值

```
sess = tf.InteractiveSession()

w = tf.Variable(10)
w.initializer.run()
print(w.eval())

w.assign(100)
# w.initializer.run() 即使调用初始化，也无法成功赋值
print(w.eval())

assign_op = w.assign(100)
# w.initializer.run() 调用与否，都不影响赋值
assign_op.eval()
print(w.eval())

sess.close()
```

### 命名作用域

可以让一部分变量拥有统一的名字

```python
sess = tf.InteractiveSession()

with tf.name_scope('name_scope'):
    var_a = tf.Variable(0, dtype=tf.int32)
    var_b = tf.Variable([1, 2], dtype=tf.float32)
with tf.variable_scope('var_scope'):
    var_c = tf.Variable(0, dtype=tf.int32)
    var_d = tf.Variable([1, 2], dtype=tf.float32)
    
print(var_a.name)
print(var_b.name)
print(var_c.name)
print(var_d.name)

sess.close()
```

```
name_scope/Variable:0
name_scope/Variable_1:0
var_scope/Variable:0
var_scope/Variable_1:0
```

## Op

operation，即操作符

### 基础操作符

```
>>> a = tf.constant(20)
>>> b = tf.constant(10)
>>> c = tf.add(a,b)
>>> d = tf.subtract(a,b)
>>> e = tf.multiply(a,b)
>>> f = tf.divide(a,b)
>>> h = tf.mod(a,b)
```

### 数值类型转换

```
 a_float = tf.cast(a,dtype=tf.float32)
```

### 初等函数

```python
# sin(a)
i = tf.sin(a_float)
# exp(1/a)
j = tf.exp(tf.divide(1.0,a_float))
# i + log(i)
k = tf.add(i,tf.log(i))
```

## Mat

tensorflow的矩阵运算

创建矩阵，使用reshape这个函数，能够方便的创建任意维度的矩阵

```python
>>> mat_a = tf.constant([1,2,3,4])
>>> sess.run(mat_a)
array([1, 2, 3, 4])
>>> mat_b = tf.reshape(mat_a,(2,2))
>>> sess.run(mat_b)
array([[1, 2],
       [3, 4]])
```

```python
>>> mat_c = tf.constant([1,3,5,7,9,11])
>>> mat_d = tf.reshape(mat_c,(2,3))
>>> mat_e = tf.matmul(mat_b,mat_d) #矩阵乘法
>>> sess.run(mat_e)
array([[15, 21, 27],
       [31, 45, 59]])
```

## Random

随机化，标准正态分布随机

```
>>> rand_normal = tf.random_normal((),mean=0.0,stddev=1.0,dtype=tf.float32,seed=None)
>>> rand_normal
<tf.Tensor 'random_normal:0' shape=() dtype=float32>
>>> sess.run(rand_normal)
-1.398341
>>> type(sess.run(rand_normal))
<class 'numpy.float32'>
```

truncated 正态随机

均匀分布随机

```
>>> truncated_normal = tf.truncated_normal((),mean=0.0,stddev=1.0,dtype=tf.float32,seed=None)
>>> rand_uniform = tf.random_uniform((),minval=0.0,maxval=1.0,dtype=tf.float32,seed=None)
```

## 占位符(placeholder)

图的节点可以没有具体值，所以出现了placeholder

```python
sess = tf.InteractiveSession()

# 长为3的向量
a = tf.placeholder(tf.float32, shape=[3])
# 1 x 2的矩阵
b = tf.placeholder(tf.bool, shape=[1, 2])

print(sess.run(a, feed_dict={a: [1, 2, 3]}))
print(sess.run([a, b], feed_dict={a: [1, 2, 3], b: [[True, False]]}))

sess.close()
```

```
[1. 2. 3.]
[array([1., 2., 3.], dtype=float32), array([[ True, False]])]
```

## Graph

### 图的获取

```python
sess = tf.InteractiveSession()

# 两个常量
tf.constant(23, dtype=tf.int32)
age = tf.constant(30, dtype=tf.int32)
# 长为3的向量
a = tf.placeholder(tf.float32, shape=[3], name="ffzdata")
# 1 x 2的矩阵
b = tf.placeholder(tf.bool, shape=[1, 2])
# 上一个存在了图中，但是python中的引用丢失了
b = tf.placeholder(tf.bool, shape=[2, 4])

g = tf.get_default_graph()
for op in g.get_operations():
    print(op.name)

sess.close()
```

```
Const
Const_1
ffzdata
Placeholder
Placeholder_1
```

### 图获取变量

使用图可以通过名字，很方便获取到变量

需要注意"myname:0" 这个“:0”代表这个变量存储在哪一个GPU上，此处代表第一个GPU

```
sess = tf.InteractiveSession()

a = tf.constant("hello", name="myname")
graph = tf.get_default_graph()
another_a = graph.get_tensor_by_name("myname:0")
print(sess.run(another_a))
b = tf.constant("nihao", name="yourname")
another_b = graph.get_tensor_by_name("yourname:0")
print(sess.run(another_b))

sess.close()
```

### 图的作用域

```python
sess = tf.InteractiveSession()

a = tf.constant(10, name="a")
# 构造一个新图A
g1 = tf.Graph()
# 查看图A的id
print('g1:', g1)
# 查看默认图id
print('default_graph:', tf.get_default_graph())

a1 = tf.constant(15, dtype=tf.int32, name="a1")
# 设置新的默认图
with g1.as_default():
    a2 = tf.constant(20, name='a2')

a3 = tf.constant(25, dtype=tf.int32, name="a3")
print("----")
# 查看该tensor的图
print(a.graph)
print(a1.graph)
print(a2.graph)
print(a3.graph)

sess.close()
```

```
g1: <tensorflow.python.framework.ops.Graph object at 0x000002B373DE8688>
default_graph: <tensorflow.python.framework.ops.Graph object at 0x000002B36B770488>
----
<tensorflow.python.framework.ops.Graph object at 0x000002B36B770488>
<tensorflow.python.framework.ops.Graph object at 0x000002B36B770488>
<tensorflow.python.framework.ops.Graph object at 0x000002B373DE8688>
<tensorflow.python.framework.ops.Graph object at 0x000002B36B770488>
```



### 图的可视化

```
g = tf.get_default_graph()
# 获取图文件的命令 这里我使用了绝对路径
tf.summary.FileWriter('D:\\Storage\\pycharmProjects\\demo', g)
```

切换到命令行，切换到与图文件相同的路径

```
> tensorboard --logdir=.
```

## 相关练习

构造一个数值占位符, 当喂入的元素是1, 2, 4, 8时, 输出这个占位符的平方

```
sess = tf.InteractiveSession()

# 构造图
x = tf.placeholder(tf.float32)
square = tf.square(x)

# 运行图
arr = [1, 2, 4, 8]
for i in arr:
    print(sess.run(square, feed_dict={x: i}))

sess.close()
```

```
1.0
4.0
16.0
64.0
```

