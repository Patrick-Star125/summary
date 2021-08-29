发现这俩东西具体学效率太低，直接贴代码和网页算了

## TensorFlow-v2.1.0

**TF深度卷积网络实现架构**

TF的实现较为直观，以多个函数为模块，无需专门写forward代码，跟着数据流看代码就可以理解网络架构

## Pytorch-v1.7

[pytorch查看模型的参数总量、占用显存量以及flops_zhuiyuanzhongjia的博客-程序员宅基地_pytorch 查看显存占用 - 程序员宅基地 (cxyzjd.com)](https://www.cxyzjd.com/article/zhuiyuanzhongjia/107230928)

pytorch模型架构的更为系统一些，能够利用到python的很多特性

```python
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

dataset = Classification_Dataloader(data)
train_dataset = DataLoader(dataset, batch_size=batch, shuffle=True, collate_fn=classification_collate)

model = Classification().to(device)
model.train(mode=True)

criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=learning_rate)

total_step = len(train_dataset)
        for epoch in range(num_epoches):
            for i, hands in enumerate(train_dataset):
                hand, label = hands
                hand = hand.to(device)
                label = label.to(device)

                optimizer.zero_grad()
                outputs = model(hand)
                outputs = outputs.unsqueeze(dim=0)
                loss = criterion(outputs, label)

                loss.backward()
                optimizer.step()
```

大概所有模型都有这样一段代码

```
卷积模块 nn.Conv2d(in_channels: int,
        out_channels: int,
        kernel_size: _size_2_t,
        stride: _size_2_t = 1,
        padding: _size_2_t = 0,)
```

[(9条消息) 在PyTorch中in-place operation的含义_york1996的博客-CSDN博客_inplace operation](https://blog.csdn.net/york1996/article/details/81835873)

**动态调整学习率**

[pytorch 动态调整学习率 - 超杰 (spytensor.com)](http://www.spytensor.com/index.php/archives/32/)

[ReduceLROnPlateau — PyTorch 1.9.0 documentation](https://pytorch.org/docs/stable/generated/torch.optim.lr_scheduler.ReduceLROnPlateau.html)

**实现EarlyStoping**

[Pytorch中实现EarlyStopping方法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/350982073)

**Pytorch深度卷积网络实现架构**

[Pytorch 中的 forward理解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/357021687)

Pytorch的网络架构实现以类为中心，需要有专门的forward代码，并且模型训练时，不需要调用forward这个函数，具体原理看上面的链接

## Pytorch与Tensorflow区别

* 在架构网络模型时，两者的代码逻辑和API调用逻辑都有很大区别，但是可以对标记忆学习
