（2009）

# 简介

Graph Cuts ，Lazy Snapping 算法的大致情况



# 简单介绍random walks算法

同前面文章大同小异



## 边的权重

使用sigmoid 函数改进得到

$G_{i,j} = \frac{2}{1+\exp(\beta||\vec c_i-\vec c_j||)}$



## alpha mask

$\alpha [x,y] = \begin{cases} 1&P[x,y] \ge 0.5 \\0&P[x,y]<0.5\end{cases}$



### Scale-space(尺度空间)的random walks

random walks算法最主要的问题是，遇到噪声或纹理，概率会变得有噪音和错误。于是造成了需要更粗糙的透明度遮罩和较不精确的边界。这是因为，如果给一条从种子点到非种子点的最短路径，对每个非种子点概率在这条路径上是递减的。

![](https://github.com/freyakniglty/SegmentationNotes/blob/master/image/lowfunctional.png)

在上图。我们需要的是fig.1 或者是fig.2(a)中的分割。但往往由于上述问题分割成不平滑的fig.2(b)



### 创建一个尺度空间

尺度空间滤波器是一个多分辨率信号分析的格式，这里的图像或者说型号是被等距高斯核心（isometric Gaussian kernel）模糊的。

中心思想是，把低等级的或者说精细的特征逐渐抹掉，直到只剩下大尺度的特征。

这是因为，在粗糙尺度上，图像消除了噪声和精细的纹理。

$S = \{ i[x,y]*f(x,y|\sigma_n):\sigma_n = 0,1,2,...,2^{N-1}\}$

这里的图像$I[x,y]$ 被算子$f(x,y|\sigma_n)$做卷积。

$f(x,y|\sigma_n) = \frac{1}{\sqrt{2\pi\sigma^2_n}}\exp \{\frac{-(x^2+y^2)}{2\sigma^2_n}\}$



我们可以增强random walks 的图像，创建6个连接的图像堆叠在一起。如下图

![](https://github.com/freyakniglty/SegmentationNotes/blob/master/image/SSRW.png)

通过对每一层的上下连接，我们可以将没有噪音的粗糙尺度和有噪音的但是更加精确的尺度进行交互。这样就能保证细节的表达以及减少噪声和纹理对分割的影响。



## 概率

在不同图层上实现random walks 算法，得到N个概率图像。我们的方法是结合每一个层次的图像。

所以有，

$P = \{P[x,y|n]:n=0,1,2,...,N-1\}$

求概率的几何平均值对每一个像素点，

$P[x,y] = (\prod^{N-1}_{n=0}P[x,y|n])^{1/N}$



# 结果

见文献
