---
layout: post
title: "PyTorch Model Storage and Use"
date: "2021-09-24 23：09：00"
tag: PyTorch python
---
- [PyTorch模型保存与使用](#orgf2c5c9d)
  - [模型的保存](#org6809e11)
  - [模型的使用](#orgd1de3b6)


<a id="orgf2c5c9d"></a>

# PyTorch模型保存与使用

将训练好的模型保存下来，这样在需要的时候就可以直接调用。


<a id="org6809e11"></a>

## 模型的保存

保存模型有两种方法，一种是保存模型的参数，再次加载时需要原有模型的信息。 另一种是保存模型的所有信息，可以直接加载为原有模型。需要注意的是，如果原有模型是在GPU上训练的，那么在加载到只有CPU 的电脑上时，需要在 torch.load() 时，加上 map_location=torch.device('cpu') 参数变换模型参数到CPU 上。

-   保存模型的所有参数

```python
#保存
torch.save(model, PATH)
#读取
use_model = torch.load(PATH)
```

-   仅保存模型的参数

```python
#保存
torch.save(CNN.state_dict(), 'CNN.pt')
#读取
new_model = CNN()
new_model.load_state_dict(torch.load('CNN.pt'))
```


<a id="orgd1de3b6"></a>

## 模型的使用

-   先看一下网络结构，使用 MNIST 数据集进行训练。并使用 torch.save(CNN.state_dict(), 'CNN.pt') 保存网络信息。

```python
from torch import nn

class CNN(nn.Module):
    def __init__(self):
	super(CNN, self).__init__()
	self.layer1 = nn.Sequential(
	    nn.Conv2d(1, 16, kernel_size=3),
	    nn.BatchNorm2d(16),
	    nn.ReLU(inplace=True))

	self.layer2 = nn.Sequential(
	    nn.Conv2d(16, 32, kernel_size=3),
	    nn.BatchNorm2d(32),
	    nn.ReLU(inplace=True),
	    nn.MaxPool2d(kernel_size=2, stride=2))

	self.layer3 = nn.Sequential(
	    nn.Conv2d(32, 64, kernel_size=3),
	    nn.BatchNorm2d(64),
	    nn.ReLU(inplace=True))

	self.layer4 = nn.Sequential(
	    nn.Conv2d(64, 128, kernel_size=3),
	    nn.BatchNorm2d(128),
	    nn.ReLU(inplace=True),
	    nn.MaxPool2d(kernel_size=2, stride=2))

	self.fc = nn.Sequential(
	    nn.Linear(128 * 4 * 4, 1024),
	    nn.ReLU(inplace=True),
	    nn.Linear(1024, 128),
	    nn.ReLU(inplace=True),
	    nn.Linear(128, 10))

    def forward(self, x):
	x = self.layer1(x)
	x = self.layer2(x)
	x = self.layer3(x)
	x = self.layer4(x)
	x = x.view(x.size(0), -1)
	x = self.fc(x)
	return x

```

-   使用训练好的网络。

在使用图片测试网络时，要先将读取的图片转换为 Tensor，之后根据网络的输入结构变换输入。 测试的CNN 网络训练时输入为 [128,1,28,28] 的 Tensor。其中128为 batch_size, 1为图像的厚度，表示图片是灰度图。[28,28] 是图片的长和宽。在用单张图片测试网络的时候，需要把图片的张量换为四维的，可以通过 unsqueeze() 函数在前面扩充一维。之后就可以直接输入到网络，并获取预测结果。

```python
newModel = CNN()
newModel.load_state_dict(torch.load('CNN.pt'))

img = Image.open('./0-label-7.png')
turn = transforms.ToTensor()
img = turn(img)
img = torch.unsqueeze(img, 0)

out = newModel(img)
_, pred = torch.max(out, 1)
print(pred)
```

