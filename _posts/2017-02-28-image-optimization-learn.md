---
layout: post
title: 计算机视觉之最优化与随机梯度下降
date: 2017-2-28
categories: blog
tags: [计算机视觉]
description: 计算机视觉
---


在上一节中，我们介绍了图像分类任务中的两个关键部分：          

- 用于把原始像素信息映射到不同类别得分的得分函数/score function
- 用于评估参数W效果(评估该参数下每类得分和实际得分的吻合度)的损失函数/loss function

其中对于线性SVM，我们有：      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter3/p1.png)  

在取到合适的参数W的情况下，我们根据原始像素计算得到的预测结果和实际结果吻合度非常高，这时候损失函数得到的值就很小。

这节我们就讲讲，怎么得到这个合适的参数W，使得损失函数取值最小化。也就是最优化的过程。

**损失函数可视化**                            

我们在计算机视觉中看到的损失函数，通常都是定义在非常高维的空间里的(比如CIFAR-10的例子里一个线性分类器的权重矩阵W是10 x 3073维的，总共有30730个参数)，人要直接『看到』它的形状/变化是非常困难的。但是机智的同学们，总是能想出一些办法，把损失函数在某种程度上可视化的。比如说，我们可以把高维投射到一个向量/方向(1维)或者一个面(2维)上，从而能直观地『观察』到一些变化。    

举个例子说，我们可以对一个权重矩阵W(例如CIFAR−10中是30730个参数)，可以找到W维度空间中的一条直线，然后沿着这条线，计算一下损失函数值的变化情况。具体一点说，就是我们找到一个方向W1(维度要和W一样，才能表示W的维度空间的一个方向/一条直线)，然后我们给不同的a值，计算L(W+aW1)，这样，如果a取得足够密，其实我们就能够在一定程度上描绘出损失函数沿着这个方向的变化了。

同样，如果我们给两个方向W1和W2，那么我们可以确定一个平面，我们再取不同值的a和b，计算L(W+aW1+bW2)的值，那么我们就可以大致绘出在这个平面上，损失函数的变化情况了。

根据上面的方法，我们画出了下面3个图。最上面的图是调整a的不同取值，绘出的损失函数变化曲线(越高值越大)；中间和最后一个图是调整a与b的取值，绘出的损失函数变化图(蓝色表示损失小，红色表示损失大)，中间是在一个图片样本上计算的损失结果，最下图为100张图片上计算的损失结果的一个平均。显然沿着直线方向得到的曲线底端为最小的损失值点，而曲面呈现的碗状图形碗底为损失函数取值最小处。 

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter3/p2.png)       

我们从数学的角度，来尝试解释一下，上面的凹曲线是怎么出来的。对于第i个样本，我们知道它的损失函数值为： 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter3/p3.png)   

插几句题外话，从之前碗状结构的示意图，你可能会猜到SVM损失函数是一个凸函数，而对于凸函数的最小值求解方法有很多种。但之后当我们把损失函数f扩充到神经网络之后，损失函数将变成一个非凸函数，而如果依旧可视化的话，我们看到的将不再是一个碗状结构，而是凹凸不平的。

关于如何高效地最小化凸函数的论文有很多，你也可以学习斯坦福大学关于（[凸函数最优化](http://stanford.edu/~boyd/cvxbook/)）的课程。    

#### 最优化

在我们现在这个问题中，所谓的『最优化』其实指的就是找到能让损失函数最小的参数W。如果大家看过或者了解凸优化的话，我们下面介绍的方法，对你而言可能太简单了，有点原始，但是大家别忘了，我们后期要处理的是神经网络的损失函数，那可不是一个凸函数哦，所以我们还是一步步来一起看看，如果去实现最优化问题。

**策略1：随机搜寻(不太实用)**            
以一个笨方法开始，我们知道，当我们手头上有参数W后，我们是可以计算损失函数，评估参数合适程度的。所以最直接粗暴的方法就是，我们尽量多地去试参数，然后从里面选那个让损失函数最小的，作为最后的W。代码当然很简单，如下：

```
# 假设 X_train 是训练集 (例如. 3073 x 50,000)
# 假设 Y_train 是类别结果 (例如. 1D array of 50,000)

bestloss = float("inf") # 初始化一个最大的float值
for num in xrange(1000):
  W = np.random.randn(10, 3073) * 0.0001 # 随机生成一组参数
  loss = L(X_train, Y_train, W) # 计算损失函数
  if loss < bestloss: # 比对已搜寻中最好的结果
    bestloss = loss
    bestW = W
  print 'in attempt %d the loss was %f, best %f' % (num, loss, bestloss)

# prints:
# in attempt 0 the loss was 9.401632, best 9.401632
# in attempt 1 the loss was 8.959668, best 8.959668
# in attempt 2 the loss was 9.044034, best 8.959668
# in attempt 3 the loss was 9.278948, best 8.959668
# in attempt 4 the loss was 8.857370, best 8.857370
# in attempt 5 the loss was 8.943151, best 8.857370
# in attempt 6 the loss was 8.605604, best 8.605604
# ... (trunctated: continues for 1000 lines)
```

一通随机试验和搜寻之后，我们会拿到试验结果中最好的参数W，然后在测试集上看看效果：

```
# 假定 X_test 为 [3073 x 10000], Y_test 为 [10000 x 1]
scores = Wbest.dot(Xte_cols) # 10 x 10000, 计算类别得分
# 找到最高得分作为结果
Yte_predict = np.argmax(scores, axis = 0)
# 计算准确度
np.mean(Yte_predict == Yte)
# 返回 0.1555
```

随机搜寻得到的参数W，在测试集上的准确率为15.5%，总共10各类别，我们不做任何预测只是随机猜的结果应该是10%，好像稍高一点，但是…大家也看到了…这个准确率…实在是没办法在实际应用中使用。


