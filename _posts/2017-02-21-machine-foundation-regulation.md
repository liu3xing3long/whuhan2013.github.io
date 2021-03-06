---
layout: post
title: 机器学习基石之正则化
date: 2017-2-21
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

**1，正规化：Regularization**           

发生overfitting 的一个重要原因可能是假设过于复杂了，我们希望在假设上做出让步，用稍简单的模型来学习，避免overfitting。例如，原来的假设空间是10次曲线，很容易对数据过拟合；我们希望它变得简单些，比如w 向量只保持三个分量（其他分量为零）。

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p1.jpg)     
图中的优化问题是NP-Hard 的。如果对w 进行更soft/smooth 的约束，可以使其更容易优化：         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p2.jpg)  

我们将此时的假设空间记为H(C)，这是“正则化的假设空间”。

**2，Weight Decay Regularization**       

通过前面的分析，我们已经把优化问题变为：
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p3.jpg) 
接下来是通过一些几何解释，用lambda 替换常数C，便于优化问题的描述和求解。这个说起来很绕，就不多说了，以免误导各位。其实，这只是林轩田解释regularization 的一种方式，其他课程不一定从这个角度进行讲解的，这里模糊的话不必深究。个人觉得lambda 的表示方式本身就很直观了。 :-)
最后得到的优化目标是：
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p4.jpg) 

lambda 的大小对于拟合的影响，一个直观例子：
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p5.jpg) 
总之，lambda 越大，对应的常数C 越小，模型越倾向于选择更小的w 向量。
这种正规化成为 weight-decay regularization，它对于线性模型以及进行了非线性转换的线性假设都是有效的

**3，正规化与VC 理论**        

根据VC Bound 理论，Ein 与 Eout 的差距是模型的复杂度。也就是说，假设越复杂（dvc 越大），Eout 与 Ein 相差就越大，违背了我们学习的意愿。
对于某个复杂的假设空间H，dvc 可能很大；通过正规化，原假设空间变为正规化的假设空间H(C)。与H 相比，H(C) 是受正规化的“约束”的，因此实际上H(C) 没有H 那么大，也就是说H(C) 的VC维比原H 的VC维要小。因此，Eout 与 Ein 的差距变小。:-)


**4，泛化的正规项 (General Regularizers)**      

指导我们更好地设计正规项的原则：target-dependent, plausible, friendly.
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p6.jpg) 
L2 and L1 Regularizer:        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p7.jpg) 

lambda 当然不是越大越好！选择合适的lambda 也很重要，它收到随机噪音和确定性噪音的影响。        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p8.jpg) 

