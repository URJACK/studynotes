# Package

## python的导入包机制

在下面这种文件结构下，如果直接运行demo.py，会出现导入包错误的问题

![image-20200716154502302](D:\LearningNotes\ai\python\1_1.png)

尽管config.py确实就在demo.py的上级目录中，但是python的解释器只载入了demo.py这**单个文件**，对于解释器来说，它的上级目录的其他文件是没有被载入的。那么如何让解释器将工程中的所有python文件全部读入进去，且符合我们的目录结构呢？

### 方案1：main.py

在项目同级目录中，新增main.py的文件，让这个main.py来引入下级目录

**main.py**

```python
import project.demos.demo
```

有了main.py之后，我们只需要运行python main.py即可运行demo.py，并且demo.py中蕴含了包结构

### 方案2：python -m

直接在控制台下，运行python -m 指定的python文件，那么其他的python文件就会被当做目录结构载入进去

**console**

```python
(py37) D:\Storage\pycharmProjects\pythonLearn>python -m project.demos.demo
```

