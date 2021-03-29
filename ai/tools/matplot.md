# matplot

## 引入依赖

```
matplotlib               2.2.5
```

## 代码导入依赖

引入pyplot画图工具

```python
import matplotlib.pyplot as plt
```

引入样式，让画图更好看

```
from matplotlib import style
```

## 中文乱码解决

```python
# 在我的 notebook 里，要设置下面两行才能显示中文
plt.rcParams['font.family'] = ['sans-serif']
# 如果是在 PyCharm 里，只要下面一行，上面的一行可以删除
plt.rcParams['font.sans-serif'] = ['SimHei']
```

## 实例代码-折线图(plot)

### Demo1-基础图

具有基础的标题、横纵坐标值、轴的名称

```python
def main():
    plt.plot([5, 2, 7], [2, 6, 11])
    plt.title("Demo Title")
    plt.xlabel("x axis")
    plt.ylabel("y axis")
    plt.show()
```

---

![image-20200815094942316](.\matplot\image-20200815094942316.png)

### Demo2-带有样式的方格图

使用pycharm的时候，按住Ctrl去点击**label**和**linewidth**的时候，会查找不到参数定义，但实际上这两个参数都是起作用的。

```python
def main():
    style.use('ggplot')
    x = [5, 8, 10]
    y = [12, 16, 8]
    x2 = [7, 9, 12]
    y2 = [14, 15, 9]
    plt.plot(x, y, color='g', label='line one', linewidth=3)
    plt.plot(x2, y2, color='r', label='line two', linewidth=5)
    plt.title('Demo2')
    plt.ylabel('Y axis')
    plt.xlabel('X axis')
    plt.legend() # 会出现每根线的label 在图的右上角
    plt.grid(True, color='k')
    plt.show()
```

---

![image-20200815100238806](.\matplot\image-20200815100238806.png)

可以对比一下不使用style.use('ggplot')的效果

![image-20200815100540284](.\matplot\image-20200815100540284.png)

## 实例代码-条形图(bar)

### Demo1-基础图

与折线图非常类似，但是使用的API不再是plot，而是bar

值得注意的是，条形图中，数据传入的横坐标x，矩形的左边等于这个x，矩形的右边的等于x+width

```python
def main():
    plt.bar([0.25, 1.25, 2.25, 3.25, 4.25], [50, 40, 70, 80, 20], label="BYD", color='b', width=.5)
    plt.bar([.75, 1.75, 2.75, 3.75, 4.75], [80, 20, 20, 50, 60], label="HONDA", color='r', width=.5)
    plt.legend()
    plt.xlabel('Days')
    plt.ylabel('Distance (kms)')
    plt.title('Information')
    plt.show()
```

---

![image-20200815100920880](.\matplot\image-20200815100920880.png)

## 实例代码-统计条形图(hist)

### Demo1-基础图

```python
def main():
    population_age = [22, 55, 62, 45, 21, 22, 34, 42, 42, 4, 2, 102, 95, 85, 55, 110, 120, 70, 65, 55, 111, 115, 80, 75, 65, 54, 44, 43, 42, 48]
    bins = [0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 110]
    plt.hist(population_age, bins, histtype='bar', color='b', rwidth=0.8)
    plt.xlabel('age groups')
    plt.ylabel('Number of people')
    plt.title('Histogram')
    plt.show()
```

---

![image-20200815104401304](.\matplot\image-20200815104401304.png)

## 实例代码-散点图(scatter)

### Demo1-基础图

```python
def main():
    x = [1, 1.5, 2, 2.5, 3, 3.5, 3.6]
    y = [7.5, 8, 8.5, 9, 9.5, 10, 10.5]
    x1 = [8, 8.5, 9, 9.5, 10, 10.5, 11]
    y1 = [3, 3.5, 3.7, 4, 4.5, 5, 5.2]
    plt.scatter(x, y, color="r", label="redTeam")
    plt.scatter(x1, y1, color="b", label="blueTeam")
    plt.xlabel("xPosition")
    plt.ylabel("yPosition")
    plt.title("counting map")
    plt.legend()
    plt.show()
```

---

![image-20200815104950401](.\matplot\image-20200815104950401.png)

## 实例代码-折线面积图(stackplot)

### Demo1-基础图

```python
def main():
    days = [1, 2, 3, 4, 5]
    sleeping = [7, 8, 6, 11, 7]
    eating = [2, 3, 4, 3, 2]
    working = [7, 8, 7, 2, 2]
    playing = [8, 5, 7, 8, 13]
    # 为了调用 legend 可以显示一些颜色对应的标签信息
    plt.plot([], [], color='m', label='Sleeping', linewidth=5)
    plt.plot([], [], color='c', label='Eating', linewidth=5)
    plt.plot([], [], color='r', label='Working', linewidth=5)
    plt.plot([], [], color='k', label='Playing', linewidth=5)
    plt.stackplot(days, sleeping, eating, working, playing, colors=['m', 'c', 'r', 'k'])
    plt.xlabel('x')
    plt.ylabel('y')
    plt.title('Stack Plot')
    plt.legend()
    plt.show()
```

---

![image-20200815111047323](.\matplot\image-20200815111047323.png)

## 实例代码-饼状面积图(pie)

### Demo1-基础图

```python
def main():
    slices = [7, 2, 6, 13]
    activities = ['sleeping', 'eating', 'working', 'playing']
    cols = ['c', 'm', 'r', 'b']
    plt.pie(slices, labels=activities, colors=cols, startangle=90, shadow=True, explode=(0, 0.1, 0, 0),
            autopct='%1.1f%%')
    plt.title('Pie Plot')
    plt.show()
```

---

![image-20200815111413541](.\matplot\image-20200815111413541.png)

## 实例代码-多图绘制(subplot)

subplot有三个参数，(总共的行数numRows，总共的列数numCols，当前切换到第几个子图plotNum)

并且，如果rows,cols,index 均小于 10。可以直接传入一个三位十进制整数。

### Demo1-基础图

f(t)的函数表达式如下所示
$$
f(t)=e^{-t}*cos(2πt)
$$

```python
def f(t):
    return np.exp(-t) * np.cos(2 * np.pi * t)


def main():
    t1 = np.arange(0.0, 5.0, 0.1)
    t2 = np.arange(0.0, 5.0, 0.02)
    plt.subplot(321)
    plt.plot(t1, f(t1), 'bo', t2, f(t2))
    plt.subplot(322)
    plt.plot(t2, np.cos(2 * np.pi * t2))
    plt.show()
```

---

![image-20200815112645226](.\matplot\image-20200815112645226.png)