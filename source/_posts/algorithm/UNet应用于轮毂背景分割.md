---
title: 'UNet应用于轮毂背景分割'
date: 2019-12-01 20:03:26
tags: [深度学习]
categories: algorithm
published: true
hideInList: false
feature: 
isTop: false
---
# 背景介绍

## 语义分割

图像语义分割(Semantic Segmentation)是指让计算机根据图像的语义来进行分割。语义分割有很多应用，例如在自动驾驶领域如何去识别汽车、行人、信号灯、车道线等具有特殊意义的道路标志是实现自动驾驶的前提。同时在OCR即文字的识别上也有广泛的应用。

<!-- more -->

## UNet

UNet网络是一种首先应用在医学上，识别细胞等组织结构的一种深度学习网络，因为其效果较好，故在分割图像上应用很多，UNet自从2015年问世以来，已经成为多种场合的使用网络其中包括地质勘探，深海云层分析等，本文首次实现将UNet应用在汽车轮毂的背景去除，经过40张标记模型的训练其效果较好，故在此分享。

# 代码分析

## 数据集处理

```python
from torch.utils.data import Dataset
import PIL.Image as Image
import os
# 从路径中读入图片
def make_dataset(root):
    imgs=[]
    n=40
    root1 = root + '/train'
    root2 = root + '/mark'
    for i in range(1,n+1):
        img=os.path.join(root1,"%03d.png"%i)
        mask=os.path.join(root2,"%03d.png"%i)
        imgs.append((img,mask))
    return imgs

class LiverDataset(Dataset):
    def __init__(self, root, transform=None, target_transform=None):
        imgs = make_dataset(root)
        self.imgs = imgs
        self.transform = transform
        self.target_transform = target_transform

    def __getitem__(self, index):
        x_path, y_path = self.imgs[index]
        img_x = Image.open(x_path)
        img_y = Image.open(y_path)
        if self.transform is not None:
            img_x = self.transform(img_x)
        if self.target_transform is not None:
            img_y = self.target_transform(img_y)
        return img_x, img_y

    def __len__(self):
        return len(self.imgs)

```

对于一般的语义识别数据集处理主要分为两个部分，分别是原始图片的处理，标签的生成及输入。对于原始图片由于现成的汽车轮毂图片并没有相对应的标签，第一步是使用photoshop等软件将原始图片的数据转为背景和本体分别为黑白的标记图及其灰度图的灰度值分别为255和0，并将这些原始图片和标签图片的名称相对应以便使用Dataloader读入。
在这里主要是要是用transfer 函数进行数据集的转换其中包括，图片大小的缩放，对图片的随机剪裁和归一化处理等，主要是为了加强数据集的数据效果。其中主要的图片部分包括，这一部分在以后可以详细了解。

## UNet网络

```python
import torch
from torch import nn

class DoubleConv(nn.Module): # 构建两重卷积的方法
    def __init__(self, in_ch, out_ch):
        super(DoubleConv, self).__init__()
        self.conv = nn.Sequential(
            nn.Conv2d(in_ch, out_ch, 3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
            nn.Conv2d(out_ch, out_ch, 3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True)
        )

    def forward(self, input):
        return self.conv(input)


class Unet(nn.Module):
    def __init__(self,in_ch,out_ch):
        super(Unet, self).__init__()

        self.conv1 = DoubleConv(in_ch, 64)
        self.pool1 = nn.MaxPool2d(2)
        self.conv2 = DoubleConv(64, 128)
        self.pool2 = nn.MaxPool2d(2)
        self.conv3 = DoubleConv(128, 256)
        self.pool3 = nn.MaxPool2d(2)
        self.conv4 = DoubleConv(256, 512)
        self.pool4 = nn.MaxPool2d(2)
        self.conv5 = DoubleConv(512, 1024)
        self.up6 = nn.ConvTranspose2d(1024, 512, 2, stride=2)
        self.conv6 = DoubleConv(1024, 512)
        self.up7 = nn.ConvTranspose2d(512, 256, 2, stride=2)
        self.conv7 = DoubleConv(512, 256)
        self.up8 = nn.ConvTranspose2d(256, 128, 2, stride=2)
        self.conv8 = DoubleConv(256, 128)
        self.up9 = nn.ConvTranspose2d(128, 64, 2, stride=2)
        self.conv9 = DoubleConv(128, 64)
        self.conv10 = nn.Conv2d(64,out_ch, 1)

    def forward(self,x):
        c1=self.conv1(x)
        p1=self.pool1(c1)
        c2=self.conv2(p1)
        p2=self.pool2(c2)
        c3=self.conv3(p2)
        p3=self.pool3(c3)
        c4=self.conv4(p3)
        p4=self.pool4(c4)
        c5=self.conv5(p4)
        up_6= self.up6(c5)
        merge6 = torch.cat([up_6, c4], dim=1)
        c6=self.conv6(merge6)
        up_7=self.up7(c6)
        merge7 = torch.cat([up_7, c3], dim=1)
        c7=self.conv7(merge7)
        up_8=self.up8(c7)
        merge8 = torch.cat([up_8, c2], dim=1)
        c8=self.conv8(merge8)
        up_9=self.up9(c8)
        merge9=torch.cat([up_9,c1],dim=1)
        c9=self.conv9(merge9)
        c10=self.conv10(c9)
        #out = nn.Sigmoid()(c10)
        return c10
```

UNet的网络并不复杂，主要是将其的高分辨率部分和低分辨率部分相结合故能取得不错的训练效果。
UNet的网络结构先对原始图像进行下采样然后进行上采样。

## 训练

```python
def train_model(model, criterion, optimizer, dataload, num_epochs=20): #num_epoch为训练的轮数
    for epoch in range(num_epochs):
        print('Epoch {}/{}'.format(epoch, num_epochs - 1))
        print('-' * 10)
        dt_size = len(dataload.dataset)
        epoch_loss = 0
        step = 0
        for x, y in dataload:
            step += 1
            inputs = x.to(device)
            labels = y.to(device)
            # zero the parameter gradients
            optimizer.zero_grad()
            # forward
            outputs = model(inputs)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()
            epoch_loss += loss.item()
            print("%d/%d,train_loss:%0.3f" % (step, (dt_size - 1) // dataload.batch_size + 1, loss.item()))
        print("epoch %d loss:%0.3f" % (epoch, epoch_loss/step)) # 输出当前轮的训练结果
        torch.save(model.state_dict(), 'weights_%d.pth' % epoch) #每训练一轮保存当前轮的训练模型
    return model

# 训练模型
def train():
    model = Unet(3, 1).to(device)  # 调入训练网络 
    batch_size = args.batch_size   # 从输入设置batch_size
    criterion = nn.BCEWithLogitsLoss()# 设置损失函数
    optimizer = optim.Adam(model.parameters()) # 优化器的方法
    liver_dataset = LiverDataset("D:/Data/yuyifenge",transform=x_transforms,target_transform=y_transforms)    # 输入数据集路径，分别对原始图片和标签进行处理
    dataloaders = DataLoader(liver_dataset, batch_size=batch_size, shuffle=True, num_workers=4)     # 从dataset中载入数据
    train_model(model, criterion, optimizer, dataloaders) #进行训练
```

这里讲图片缩放的大小为448*448同时batch_size设置为2，这样在P1000这样显卡内存为4G的机器上训练师显存能够得到充分的利用。同时由于batch_size和图片的分辨率都会对训练结构产生影响，且两者的数值都应该保持一个相对较大的值。
在显存较大的机器上进行训练师最好将图片的分辨率和batch_size设置为较大的值。

## 验证

```python
def test():
    model = Unet(3, 1)
    model.load_state_dict(torch.load(args.ckpt,map_location='cpu'))
    liver_dataset = LiverDataset("D:/Data/yuyifenge/test", transform=x_transforms,target_transform=y_transforms)
    dataloaders = DataLoader(liver_dataset, batch_size=1)
    model.eval()
    import matplotlib.pyplot as plt
    plt.ion()
    z=0
    with torch.no_grad():
        for x, _ in dataloaders:
            y=model(x)
            img_y=torch.squeeze(y).numpy()
            plt.imshow(img_y)
            img_y = img_y * 30
            # print(img_y)
            cv2.imwrite("D:/Data/yuyifenge/test/t/%03d.png"%z,img_y)
            plt.pause(0.01)
            z += 1
        plt.show()
```

验证的过程比较简单，即将模型信息导入后直接使用