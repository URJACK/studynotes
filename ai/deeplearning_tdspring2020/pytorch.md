# pytorch

## 基础

### tensor与numpy转化

```python
def main():
    x_numpy = np.array([1, 2, 3.])
    y_torch = torch.tensor([3, 4, 5.])
    y_numpy = y_torch.numpy()
    x_torch = torch.from_numpy(x_numpy)
    print("x + y")
    print(x_torch + y_torch)
    print(x_numpy + y_numpy)
    print()
```

---

```
x + y
tensor([4., 6., 8.], dtype=torch.float64)
[4. 6. 8.]
```

### 转换维度

```python
def main():
    # "MNIST"
    N, C, W, H = 10000, 3, 28, 28
    X = torch.randn((N, C, W, H))

    print(X.shape)
    print(X.view(N, C, 784).shape)
    print(X.view(-1, C, 784).shape)  # automatically choose the 0th dimension
```

---

```
torch.Size([10000, 3, 28, 28])
torch.Size([10000, 3, 784])
torch.Size([10000, 3, 784])
```

### 广播

```python
def main():
    x = torch.empty(5, 1, 4, 1)
    y = torch.empty(3, 1, 1)
    print((x + y).size())
```

---

```
torch.Size([5, 3, 4, 1])
```

### 计算图（求梯度）

![image-20200911115952876](.\pytorch\image-20200911115952876.png)

```python
def main():
    a = torch.tensor(2.0, requires_grad=True)  # we set requires_grad=True to let PyTorch know to keep the graph
    b = torch.tensor(1.0, requires_grad=True)
    c = a + b
    d = b + 1
    e = c * d
    print('c', c)
    print('d', d)
    print('e', e)
```

---

```
c tensor(3., grad_fn=<AddBackward0>)
d tensor(2., grad_fn=<AddBackward0>)
e tensor(6., grad_fn=<MulBackward0>)
```

### GPU分配

```python
def main():
    cpu = torch.device("cpu")
    gpu = torch.device("cuda")

    x = torch.rand(10)
    print(x)
    x = x.to(gpu)
    print(x)
    x = x.to(cpu)
    print(x)
```

---

```
tensor([0.5938, 0.2183, 0.9172, 0.8340, 0.8622, 0.2281, 0.3298, 0.3373, 0.7963,
        0.5875])
tensor([0.5938, 0.2183, 0.9172, 0.8340, 0.8622, 0.2281, 0.3298, 0.3373, 0.7963,
        0.5875], device='cuda:0')
tensor([0.5938, 0.2183, 0.9172, 0.8340, 0.8622, 0.2281, 0.3298, 0.3373, 0.7963,
        0.5875])
```

### 求范数

![image-20200911153533682](.\pytorch\image-20200911153533682.png)



### 求取梯度

#### 例子1

无论是简单的函数

```python
def f(x):
    return (x - 2) ** 2


def fp(x):
    return 2 * (x - 2)


def main():
    x = torch.tensor([1.0], requires_grad=True)

    y = f(x)
    y.backward()

    # 手动写的微分函数，求得的数值
    print('Analytical f\'(x):', fp(x))
    # pytorch 自动求得的微分数值
    # 注意，求微分的步骤：
    #   让计算图的后序输出tensorA调用backward方法
    #   需要求得梯度的tensorB获取其grad变量
    print('PyTorch\'s f\'(x):', x.grad)

```

---

```
Analytical f'(x): tensor([-2.], grad_fn=<MulBackward0>)
PyTorch's f'(x): tensor([-2.])
```

#### 例子2

还是复杂的函数，pytorch都能够帮我们求的微分

```python
def g(w):
    return 2 * w[0] * w[1] + w[1] * torch.cos(w[0])


def grad_g(w):
    return torch.tensor([2 * w[1] - w[1] * torch.sin(w[0]), 2 * w[0] + torch.cos(w[0])])


def main():
    w = torch.tensor([np.pi, 1], requires_grad=True)

    z = g(w)
    z.backward()

    print('Analytical grad g(w)', grad_g(w))
    print('PyTorch\'s grad g(w)', w.grad)
```

---

```
Analytical grad g(w) tensor([2.0000, 5.2832])
PyTorch's grad g(w) tensor([2.0000, 5.2832])
```

#### 例子3

```python
d = 2
n = 50


# define a linear model with no bias
def model(X, w):
    return X @ w


# the residual sum of squares loss function
def rss(y, y_hat):
    return torch.norm(y - y_hat) ** 2 / n


# analytical expression for the gradient
def grad_rss(X, y, w):
    return -2 * X.t() @ (y - X @ w) / n


def main():
    # make a simple linear dataset with some noise

    X = torch.randn(n, d)
    true_w = torch.tensor([[-1.0], [2.0]])
    y = X @ true_w + torch.randn(n, 1) * 0.1
    print('X shape', X.shape)
    print('y shape', y.shape)
    print('w shape', true_w.shape)

    w = torch.tensor([[1.], [0]], requires_grad=True)
    y_hat = model(X, w)

    loss = rss(y, y_hat)
    loss.backward()

    print('Analytical gradient', grad_rss(X, y, w).detach().view(2).numpy())
    print('PyTorch\'s gradient', w.grad.view(2).numpy())

    step_size = 0.1

    print('iter,\tloss,\tw')
    for i in range(20):
        y_hat = model(X, w)
        loss = rss(y, y_hat)

        loss.backward()  # compute the gradient of the loss

        w.data = w.data - step_size * w.grad  # do a gradient descent step

        print('{},\t{:.2f},\t{}'.format(i, loss.item(), w.view(2).detach().numpy()))

        # We need to zero the grad variable since the backward()
        # call accumulates the gradients in .grad instead of overwriting.
        # The detach_() is for efficiency. You do not need to worry too much about it.
        w.grad.detach()
        w.grad.zero_()

    print('\ntrue w\t\t', true_w.view(2).numpy())
    print('estimated w\t', w.view(2).detach().numpy())
```

---

```
X shape torch.Size([50, 2])
y shape torch.Size([50, 1])
w shape torch.Size([2, 1])
Analytical gradient [ 3.4464679 -3.108014 ]
PyTorch's gradient [ 3.4464674 -3.1080143]
iter,	loss,	w
0,	6.54,	[0.3107065 0.6216029]
1,	2.95,	[0.08559284 0.83576316]
2,	2.06,	[-0.10050935  1.0165985 ]
3,	1.44,	[-0.25436962  1.169285  ]
4,	1.01,	[-0.3815813  1.2981968]
5,	0.71,	[-0.48676625  1.4070293 ]
6,	0.50,	[-0.5737437  1.4989048]
7,	0.35,	[-0.6456699  1.5764612]
8,	0.25,	[-0.70515305  1.6419265 ]
9,	0.18,	[-0.754349   1.6971829]
10,	0.13,	[-0.7950394  1.74382  ]
11,	0.09,	[-0.8286971  1.7831802]
12,	0.07,	[-0.8565393  1.8163975]
13,	0.05,	[-0.8795725  1.8444293]
14,	0.04,	[-0.8986286  1.8680837]
15,	0.03,	[-0.9143954  1.8880436]
16,	0.02,	[-0.9274416  1.9048854]
17,	0.02,	[-0.93823737  1.9190954 ]
18,	0.02,	[-0.9471716  1.9310844]
19,	0.01,	[-0.9545658  1.9411992]

true w		 [-1.  2.]
estimated w	 [-0.9545658  1.9411992]
```

### 数据集

```python
from torch.utils.data import Dataset, DataLoader


class FakeDataset(Dataset):

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __len__(self):
        return len(self.x)

    def __getitem__(self, idx):
        return self.x[idx], self.y[idx]


def main():
    x = np.random.rand(100, 10)
    y = np.random.rand(100)

    dataset = FakeDataset(x, y)
    dataloader = DataLoader(dataset, batch_size=4,
                            shuffle=True, num_workers=4)

    for i_batch, sample_batched in enumerate(dataloader):
        print(i_batch, sample_batched)
```

---

```
0 [tensor([[0.0818, 0.8047, 0.6366, 0.4999, 0.0747, 0.2567, 0.0434, 0.0925, 0.1990,
         0.6307],
        [0.9018, 0.7323, 0.1050, 0.5108, 0.2673, 0.7904, 0.8970, 0.5259, 0.4973,
         0.7579],
        [0.9361, 0.2944, 0.0397, 0.2619, 0.9098, 0.3776, 0.7837, 0.7170, 0.7712,
         0.7568],
        [0.3523, 0.8437, 0.5326, 0.2082, 0.2677, 0.1037, 0.3203, 0.9154, 0.8982,
         0.3879]], dtype=torch.float64), tensor([0.6071, 0.5430, 0.4785, 0.1071], dtype=torch.float64)]
 ...
 ....
 24 [tensor([[0.2540, 0.8099, 0.8193, 0.0225, 0.1492, 0.2928, 0.3941, 0.2389, 0.3384,
         0.2143],
        [0.8605, 0.6706, 0.3308, 0.0088, 0.0173, 0.6444, 0.4358, 0.7347, 0.0021,
         0.3783],
        [0.8354, 0.8813, 0.3048, 0.3033, 0.2954, 0.2732, 0.3943, 0.4668, 0.4581,
         0.1431],
        [0.9747, 0.8195, 0.8178, 0.1114, 0.7405, 0.5671, 0.2157, 0.9627, 0.2010,
         0.2740]], dtype=torch.float64), tensor([0.5088, 0.7632, 0.1748, 0.2925], dtype=torch.float64)]
```



## 神经网络

### 简单的神经网络

使用pytorch定义网络时，不再需要关注具体的weight和bias的数组的形状。

在这里，也可以快速获取到网络中的参数

```python
def main():
    d_in = 3
    d_out = 4
    linear_module = torch.nn.Linear(d_in, d_out)

    example_tensor = torch.tensor([[1., 2, 3], [4, 5, 6]])
    # applys a linear transformation to the data
    transformed = linear_module(example_tensor)
    print('example_tensor', example_tensor.shape)
    print('transormed', transformed.shape)
    print()
    print('We can see that the weights exist in the background\n')
    print('W:', linear_module.weight)
    print('b:', linear_module.bias)
```

---

```
example_tensor torch.Size([2, 3])
transormed torch.Size([2, 4])

We can see that the weights exist in the background

W: Parameter containing:
tensor([[ 0.2287,  0.4232, -0.4227],
        [-0.3289, -0.3963, -0.2294],
        [ 0.4602, -0.4364,  0.2267],
        [-0.1246, -0.3679, -0.4644]], requires_grad=True)
b: Parameter containing:
tensor([ 0.4479,  0.0732, -0.1627, -0.1759], requires_grad=True)
```

### 激活函数

```python
def main():
    activation_fn = torch.nn.ReLU()  # we instantiate an instance of the ReLU module
    example_tensor = torch.tensor([-1.0, 1.0, 0.0])
    activated = activation_fn(example_tensor)
    print('example_tensor', example_tensor)
    print('activated', activated)
```

---

```
example_tensor tensor([-1.,  1.,  0.])
activated tensor([0., 1., 0.])
```

### 多层神经网络

```python
def main():
    d_in = 3
    d_hidden = 4
    d_out = 1
    model = torch.nn.Sequential(
        torch.nn.Linear(d_in, d_hidden),
        torch.nn.Tanh(),
        torch.nn.Linear(d_hidden, d_out),
        torch.nn.Sigmoid()
    )

    example_tensor = torch.tensor([[1., 2, 3], [4, 5, 6]])
    transformed = model(example_tensor)
    print('transformed', transformed.shape)
```

---

```
transformed torch.Size([2, 1])
```

### 损失函数

#### 平方差

```python
def main():
    mse_loss_fn = torch.nn.MSELoss()

    input = torch.tensor([[0., 0, 0]])
    target = torch.tensor([[1., 0, -1]])

    loss = mse_loss_fn(input, target)

    print(loss)
```

---

```
tensor(0.6667)
```

#### 交叉熵

```python
def main():
    loss = nn.CrossEntropyLoss()

    input = torch.tensor([[-1., 1], [-1, 1], [1, -1]])  # raw scores correspond to the correct class
    # input = torch.tensor([[-3., 3],[-3, 3],[3, -3]]) # raw scores correspond to the correct class with higher confidence
    # input = torch.tensor([[1., -1],[1, -1],[-1, 1]]) # raw scores correspond to the incorrect class
    # input = torch.tensor([[3., -3],[3, -3],[-3, 3]]) # raw scores correspond to the incorrect class with incorrectly placed confidence

    target = torch.tensor([1, 1, 0])
    output = loss(input, target)
    print(output)
```

---

```
tensor(0.1269)
```

### 优化器（简化梯度下降）

#### 例子1

```python
def main():
    # create a simple model
    model = torch.nn.Linear(1, 1)

    # create a simple dataset
    X_simple = torch.tensor([[1.]])
    y_simple = torch.tensor([[2.]])

    # create our optimizer
    optim = torch.optim.SGD(model.parameters(), lr=1e-2)
    mse_loss_fn = torch.nn.MSELoss()

    y_hat = model(X_simple)
    print('model params before:', model.weight)
    loss = mse_loss_fn(y_hat, y_simple)
    # 预备：这里有一步清零梯度的操作
    optim.zero_grad()
    # 第一步：计算微分，让需要梯度的参数获取到相应的grad
    loss.backward()
    # 第二步：调用优化器，让每个参数依据（grad，优化算法）来进行优化
    optim.step()
    print('model params after:', model.weight)
```

---

```
model params before: Parameter containing:
tensor([[-0.6584]], requires_grad=True)
model params after: Parameter containing:
tensor([[-0.6169]], requires_grad=True)
```

#### 例子2（代入了momentum）

原函数是蓝色的点

拟合后的图像（红线）

![image-20200911163311542](.\pytorch\image-20200911163311542.png)

```python
def main():
    # 创立原函数
    d = 1
    n = 200
    X = torch.rand(n, d)
    y = 4 * torch.sin(np.pi * X) * torch.cos(6 * np.pi * X ** 2)

    plt.scatter(X.numpy(), y.numpy())
    plt.title('plot of $f(x)$')
    plt.xlabel('$x$')
    plt.ylabel('$y$')

    plt.show()

    # 创立模型、训练模型

    step_size = 0.05
    momentum = 0.9
    n_epochs = 6000
    n_hidden_1 = 32
    n_hidden_2 = 32
    d_out = 1

    neural_network = nn.Sequential(
        nn.Linear(d, n_hidden_1),
        nn.Tanh(),
        nn.Linear(n_hidden_1, n_hidden_2),
        nn.Tanh(),
        nn.Linear(n_hidden_2, d_out)
    )

    loss_func = nn.MSELoss()

    optim = torch.optim.SGD(neural_network.parameters(), lr=step_size, momentum=momentum)
    print('iter,\tloss')
    for i in range(n_epochs):
        y_hat = neural_network(X)
        loss = loss_func(y_hat, y)
        optim.zero_grad()
        loss.backward()
        optim.step()

        if i % (n_epochs // 10) == 0:
            print('{},\t{:.2f}'.format(i, loss.item()))

    X_grid = torch.from_numpy(np.linspace(0, 1, 50)).float().view(-1, d)
    y_hat = neural_network(X_grid)
    plt.scatter(X.numpy(), y.numpy())
    plt.plot(X_grid.detach().numpy(), y_hat.detach().numpy(), 'r')
    plt.title('plot of $f(x)$ and $\hat{f}(x)$')
    plt.xlabel('$x$')
    plt.ylabel('$y$')
    plt.show()
```

---

没有代入momentum的时候，loss下降的很慢

```
iter,	loss
0,	4.06
600,	0.05
1200,	0.00
1800,	0.00
2400,	0.00
3000,	0.00
3600,	0.00
4200,	0.00
4800,	0.00
5400,	0.00
```

