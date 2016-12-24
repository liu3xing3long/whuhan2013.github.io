---
layout: post
title: 中值滤波与图像锐化
date: 2016-12-23
categories: blog
tags: [图像处理]
description: 图像处理
---

**本文主要包括以下内容**     

- 中值滤波及其改进算法
- 图像锐化， 包括梯度算子、拉普拉斯算子、高提升滤波和高斯-拉普拉斯变换
- 本章的典型囊例分析
    + 对椒盐噪声的平滑效果比较
    + Laplacian与LoG算子的锐化效果比较


#### 中值滤波     
中值滤波本质上是一种统计排序滤波器． 对于原图像中某点（i,j)， 中值滤波以该点为中
心的邻域内的所有像素的统计排序中值作为（i, j） 点的响应．                  
中值不同于均值， 是指排序队列中位于中间位置的元素的值，例如＝采用3x3 中值滤披
器， 某点。(i,j) 的8 个邻域的一系列像素值为： 12, 18, 18, 11, 23, 22, 13, 25, 118,
统计排序结果为： 1l, 12, 13, 18, 18, 22, 23, 25, 118. 排在中间位置〈第5 位〉的18
即作为(i, j）点中值滤波的响应g(i, j）. 显然， 中值滤波并非线性滤披器．     

中值滤波对于某些类型的随机噪声具有非常理想的降噪能力， 对于线性平滑滤波而言，
在处理像萦邻壤之内的噪声点时， 噪声的存在总会或多或少影响该点的像素值的计算（高斯
平滑影响的程度与噪声点到中心点的距离成正比〉，但在中值滤被中噪声点则常常直接忽略掉的：而且与线性平滑滤波器相比， 中值滤波在降噪同时引起的模糊效应较低。中值滤波的一种典型应用是清除椒盐噪声．

下面首先简单介绍一下常见的噪声模型，接着给出中值滤波的Matlab实现：     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter5/p15.png) 

Matlab提供了medfilt2函数实现中值滤波， 原型为:      
I2 = medfilt2(I1,[m,n])     

参数说明      
• I1是原因矩阵．           
• m和n是中值滤波处理的模板大小，默认3x3.       

输出结果      
输出I2是中值滤波后的图像矩阵．     

下面的程序分别给出了一幅受椒盐噪声污染的图像经过平均平滑、高斯平滑和中值撼泼
的处理效果．    

```
I = imread('lena_salt.bmp');
J = imnoise(I,'salt & pepper');
w = [1,2,1;2,4,2;1,2,1]/16;
J1 = imfilter(J,w,'corr','replicate');

w = [1,1,1;1,1,1;1,1,1]/9;
J2 = imfilter(J,w,'corr','replicate');

J3 = medfilt2(J,[3,3]);
figure;
subplot(2,3,1);
imshow(I),title('原图像');
subplot(2,3,2);
imshow(J),title('椒盐噪声');  
subplot(2,3,4);
imshow(J1),title('高斯平滑');
subplot(2,3,5);
imshow(J2),title('平均平滑');
subplot(2,3,6);
imshow(J3),title('中值平滑');
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter52/p1.png)

如图从中可见线性平滑滤波在降噪的同时不可避免地造成了模糊，而中值滤波在有效抑制椒盐噪声的同时模糊效应明显低得多，因而对于椒盐噪声污染的图像，中值滤波要远远优于线性平滑滤波．    

**一种改进的中值滤波策略**      
中值滤波效果依赖于滤波窗口的大小， 太大会使边缘模糊， 太小了则去噪效果不佳。 因为噪声点和边缘点同样是灰度变化较为剧烈的像素， 普通中值滤波在改变噪声点灰度值时，会一定程度地改变边缘像素灰度值。但是噪声点几乎都是邻域像素的极值，而边缘往往不是，因此可以利用这个特性来限制中值滤波。     

具体的改进方法如下： 逐行扫描图像， 当处理每一个像素时， 判断该像素是否是滤披窗口覆盖下邻域像素的极大或者极小值。 如果是， 则采用正常的中值滤波处理该像素：如果不是， 则不予处理。 在实践中这种方法能够非常有效地去除突发噪声点， 尤其是椒盐噪声， 且几乎不影响边缘。
由于算法可以根据局部邻域的具体情况而自行选择执行不同的操作， 因此改进的中值滤波也称为自适应中值滤波．       

自适应中值滤波对边缘进行了更好的保留。     

**中值滤波的工作原理**    
与线性平滑滤波考虑邻域中每个像素的作用不同，中值滤波在每个n×n邻域内都会忽略那些相对于邻域内大部分像素更亮或更睛，并且所占区域小于像素总数一半$(n^2/2）$的那些像素的影响，而实际上满足这样条件被忽略掉的像素往往就是噪声。      

注意：作为一种非线性滤波，中值滤波有可能会改变图像的性质，因而一般不适用于像军事图像处理、医学图像处理等领域．


### 图像锐化
图像锐化的目的是使模糊的图像变得更加清晰． 其应用广泛， 包括从医学成像到工业检
测和军事系统的制导等。          

图像锐化主要用于增强图像的灰度跳变部分，这一点与图像平滑对灰度跳变的抑制正好 相反，事实上从平滑与锐化的两种运算算子上也能说明这一点，线性平滑都是基于对图像邻域的加权求和或积分运算，而锐化则通过其逆运算导数(梯度〉或有限差分来实现。     

在讨论平滑的时候曾提到噪声和边缘都会使图像产生灰度跳变，为了在平滑时能够将噪声和边缘区别对待，5.3.5节中给出了一种自适应滤波的解决方案。同样，在锐化处理中如何区分噪声和边缘仍然是我们面临的一个课题，即在平滑处理中平滑的对象是噪声而不涉及边缘,在锐化中锐化的对象是边缘而不涉及噪声。  

#### 基于一阶导数的图像增强一一梯度算子       
回忆一下高等数学中梯度的定义，对于连续2 维函数.f(x, y），其在点(x，y）处的梯度
是下面的二维列向量：        

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter52/p2.png)
其中，$w_1$对接近正45 度边缘有较强响应： $w_2$ 对接近负45 度边缘有较强响应．   

**基于Robert交叉梯度的图像锐化**       
通过前面学习的滤波知识可知，只要分别以w1和w2为模板，对原图像进行滤波就可得到GI和G2.而根据公式5-9，最终的Robert交叉梯度图像（b）为：G = ｜G1｜ + ｜G2｜.          
在进行锐化滤波之前，我们要将图像类型从uint8转换为double.因为锐化模板计算时常常使输出产生负值， 如果采用无符号的 uint8 型， 则负位会被截断．

在调用函数 imfilter 时，还要注意不要使用默认的填充方式． 因为 Matlab 默认会在滤波时进行 “0” 填充，这会导致图像在边界处产生一个人为的灰度跳变，从而在梯度图像中产生高响应， 而这些人为高响应值的存在将导致图像中真正的边缘和其他我们关心的细节的响应在输出梯度图像中被压缩在一个很窄的灰度范围， 同时也影响显示的效果． 我们这里采用了 ’replicate’的重复填充方式， 也可采用’symmetric’的对称填充方式．     

程序实现如下：   

```
I = imread('bacteria.BMP');
temp = I;
I = double(I);
w1 = [-1 0; 0 1];
w2 = [0 -1;1 0];

G1 = imfilter(I,w1,'corr','replicate');
G2 = imfilter(I,w2,'corr','replicate');
G = abs(G1)+abs(G2);
figure;
subplot(2,2,1);
imshow(temp),title('原图像');
subplot(2,2,2);
imshow(abs(G1),[]),title('w1滤波'); 
subplot(2,2,3);
imshow(abs(G2),[]),title('w2滤波'); 
subplot(2,2,4);
imshow(G,[]),title('Robert梯度'); 
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter52/p3.png)

如图可知，w1模板对正45度左右显示较好，w2模板对－45度左右显示较好。    
注意，为便于观察效果，做了显示时的重新标定， 即将图像的灰度范围线性变换到 0-255 之内， 并使图像的最小灰度值为 0，最大灰度值为255.      
imshow(K,[])显示K，并将K的最大值和最小值分别作为纯白(255)和纯黑(0)，中间的K值映射为0到255之间的标准灰度值。    

#### Sobel梯度    
由于滤波时我们总是喜欢奇数尺寸的模板， 因而一种计算Sobel 梯度的Sobel 模板更加常用：     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter52/p4.png)
下面的Matlab 程序计算了一幅图像的竖直和水平梯度， 它们的和可以作为完整的Sobel梯度。

```
I = imread('bacteria.BMP');
w1 = fspecial('sobel');
w2 = w1';
G1 = imfilter(I,w1);
G2 = imfilter(I,w2);
G = abs(G1)+abs(G2);
figure;
subplot(2,2,1);
imshow(I),title('原图像');
subplot(2,2,2);
imshow(G1,[]),title('水平sobel');  
subplot(2,2,3);
imshow(G2,[]),title('竖直sobel');
subplot(2,2,4);
imshow(G,[]),title('sobel');
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter52/p5.png)

#### 基于二阶微分的图像增强拉普拉斯算子     
下面介绍一种对于图像锐化而言应用更为广泛的基于二阶微分的拉普拉斯（Laplacian)算子.    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter52/p6.png)

分析拉普拉斯模板的结构，可知这种模板对于90度的旋转是各向同性的。所谓对于某角
度各向同性是指把原图像旋转该角度后再进行滤波与先对原图像滤波再旋转该角度的结果相
同。这说明拉普拉斯算子对于接近水平和接近坚直方向的边缘都有很好的增强，从而也就避
免我们在使用梯度算子时要进行两次滤波的麻烦。更进一步，我们还可以得到如下对于45°
旋转各向同性的滤波器：     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter52/p7.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter52/p8.png)

分别使用上述3种拉普拉斯滤波的Matlab滤波程序如下    

```
I = imread('bacteria.BMP');
temp = I;
I = double(I);
w1 = [0 -1 0;-1 4 -1;0 -1 0];
L1 = imfilter(I,w1,'corr','replicate');
w2 = [-1 -1 -1;-1 8 -1 ;-1 -1 -1];
L2 = imfilter(I,w2,'corr','replicate');
w3 = [1 4 1;4 -20 4;1 4 1];
L3 = imfilter(I,w3,'corr','replicate');
figure;
subplot(2,2,1);
imshow(temp),title('原图像');
subplot(2,2,2);
imshow(L1);
subplot(2,2,3);
imshow(L2);
subplot(2,2,4);
imshow(L3);
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter52/p9.png)

上述程序运行结果如图5.9 所示．对于细菌图像，拉普拉斯锐化效果与之前Robert 与
Sobel梯度锐化明显不同的一点是输出图像中的双边缘。此外，我们还注意到拉普拉斯锐化似乎
对一些离散点有较强的响应，当然由于噪声也是离散点，因此这个性质有时是我们所不希望的．    

