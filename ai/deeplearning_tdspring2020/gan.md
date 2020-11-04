# GAN

## 基础

GAN的主要应用场景

![image-20200916115250147](.\gan\image-20200916115250147.png)

### 主要构成

生成器

![image-20200916115419611](.\gan\image-20200916115419611.png)

判别器

对于判别器来说，反面的例子是极为重要的，如果只有正面的例子，很有可能会将判别器训练成只会打高分的判别器，进而失去了判别的意义。

![image-20200916115523529](.\gan\image-20200916115523529.png)

生成对抗

![image-20200916115920795](.\gan\image-20200916115920795.png)

### 生成对抗

#### 训练思想

Initialize generator and discriminator.

In each training iteration:

​	step1:Fix generator G,and update discriminator D.Discriminator learns to assign high scores to real objects and low scores to generated objects.

![image-20200916120417488](.\gan\image-20200916120417488.png)

​	step2:Fix discriminator D,and update generator G. Generator learns to "fool" the discriminator.

![image-20200916120704632](.\gan\image-20200916120704632.png)

训练的方式

![image-20200916173301769](.\gan\image-20200916173301769.png)

#### GAN训练的伪代码

![image-20200916130856811](.\gan\image-20200916130856811.png)

#### GAN的线性变化

![image-20200916131431242](.\gan\image-20200916131431242.png)

#### Structured Learning

训练重要的是需要将各个component之间的关系表示清楚。

![image-20200916133200660](.\gan\image-20200916133200660.png)

![image-20200916153229323](.\gan\image-20200916153229323.png)

![image-20200916133740587](.\gan\image-20200916133740587.png)

### Auto-encoder

显然增大code的维度，对于还原的效果是具有帮助的。

![image-20200916134010159](.\gan\image-20200916134010159.png)

### FID比较不同的GAN

![image-20200916174412659](.\gan\image-20200916174412659.png)

## CGAN

![image-20200916175006614](.\gan\image-20200916175006614.png)

## Code

使用GAN生成动漫图像

### dataset.py

1·Dataset需要覆写的两个方法：

```
__getitem__ 与 __len__
```

这在之后使用数据集的时候，具有重要作用。

2·opencv库中，图片使用的是BGR格式，而非RGB格式，所以需要有一次转换

3·transform使用的是torchvision.transform



```python
from torch.utils.data import Dataset, DataLoader
import glob
import torchvision.transforms as transforms
import cv2
import os
import random
import torch
import numpy as np


class FaceDataset(Dataset):
    def __init__(self, fnames, transform):
        self.transform = transform
        self.fnames = fnames
        self.num_samples = len(self.fnames)

    def __getitem__(self, idx):
        fname = self.fnames[idx]
        img = cv2.imread(fname)
        img = self.BGR2RGB(img)  # because "torchvision.utils.save_image" use RGB
        img = self.transform(img)
        return img

    def __len__(self):
        return self.num_samples

    @staticmethod
    def BGR2RGB(img):
        return cv2.cvtColor(img, cv2.COLOR_BGR2RGB)


def get_dataset(root):
    fnames = glob.glob(os.path.join(root, '*'))
    # resize the image to (64, 64)
    # linearly map [0, 1] to [-1, 1]
    transform = transforms.Compose(
        [transforms.ToPILImage(),
         transforms.Resize((64, 64)),
         transforms.ToTensor(),
         transforms.Normalize(mean=[0.5] * 3, std=[0.5] * 3)])
    dataset = FaceDataset(fnames, transform)
    return dataset


def same_seeds(seed):
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed(seed)
        torch.cuda.manual_seed_all(seed)  # if you are using multi-GPU.
    np.random.seed(seed)  # Numpy module.
    random.seed(seed)  # Python random module.
```

