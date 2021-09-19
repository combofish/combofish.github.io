---
layout: post
title: "MNIST Dataset Download and Visualization"
date: "2021-09-19 17：17：00"
tag: mnist pytorch
---
- [MNIST 数据集下载及可视化](#orga336a9b)
  - [MNIST 介绍](#orgbce01e7)
  - [下载数据集并保存成图片](#org11d8331)


<a id="orga336a9b"></a>

# MNIST 数据集下载及可视化

最近用到了 MNIST 训练神经网络，但是下载下来的数据集默认是 `*-ubyte.gz` 的压缩包。 无法直观的看到数据本身的图片是什么样子，训练的神经网络达到99% 的正确率还是觉的一头雾水， 想看一下自己到底用的什么图片。

但是网上关于 **MNIST数据集可视化** 的文章有些比较繁琐，特在此记录将MNIST数据集下载下来，并以图片形式保存到本地的操作步骤。


<a id="orgbce01e7"></a>

## MNIST 介绍

MNIST 数据集来自美国国家标准与技术研究所, National Institute of Standards and Technology (NIST). 训练集 (training set) 由来自 250 个不同人手写的数字构成, 其中 50% 是高中学生, 50% 来自人口普查局 (the Census Bureau) 的工作人员. 测试集(test set) 也是同样比例的手写数字数据.


<a id="org11d8331"></a>

## 下载数据集并保存成图片

-   使用 torchvision 中的 datasets 下载数据集。注意不用 `transform`。

```python
from torchvision import datasets

train_data = datasets.MNIST(root="./data/", train=True, download=True)
test_data = datasets.MNIST(root="./data/", train=False, download=True)
```

-   查看数据集内容

```python
from icecream import ic

ic(len(train_data), len(test_data))
ic(train_data[0])
ic(train_data[0][0])

# RESULT
# ic| len(train_data): 60000, len(test_data): 10000
# ic| train_data[0]: (<PIL.Image.Image image mode=L size=28x28 at 0x7F94EF6E6AF0>, 5)
# ic| train_data[0][0]: <PIL.Image.Image image mode=L size=28x28 at 0x7F94EF6E6AF0>
```

可以看到， 训练集和测试集的大小分别是 60000, 和 10000 个图片，`train_data` 是由 60000 个带有图片和标号信息的元组构成， 而且图片的格式是 pillow 库的 Image。所以我们可以直接使用 PIL.Image 对象的 save() 方法保存成图片。

-   保存到图片

```python
import os

saveDirTrain = './DataImages-Train'
saveDirTest = './DataImages-Test'

if not os.path.exists(saveDirTrain):
    os.mkdir(saveDirTrain)
if not os.path.exists(saveDirTest):
    os.mkdir(saveDirTest)

def save_img(data, save_path):
    for i in tqdm(range(len(data))):
	img, label = data[i]
	img.save(os.path.join(save_path, str(i) + '-label-' + str(label) + '.png'))

save_img(train_data, saveDirTrain)
save_img(test_data, saveDirTest)
```

+ 图片结果

![下载后的图片](/images/MNIST-Dataset-Download/MNIST-Dataset.png)


-   完整代码文件可以看这里
[main.py](https://github.com/combofish/chips-get/blob/master/Python3/DeepLearning/MNIST-Download/main.py)
