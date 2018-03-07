# 二维图像的分割

## 简介

一个较好的互动分割有以下特点

- 计算速度快
- 选择编辑速度快
- 用足够的用户输入能得到任意需要的分割
- 分割是直觉上的

![](https://github.com/freyakniglty/SegmentationNotes/blob/master/image/segmentation.png
)

这是一个类似K-means的算法。有此领域前面的论文指出要寻求randomwalker到一个种子点的解，可以由解决Dirichlet带有边界条件的问题来得到。combinatorial Dirichlet 问题在任意图像上的解产生于结点的电动势能（electric potentials）在电路上的分布和边界条件。

![](https://github.com/freyakniglty/SegmentationNotes/blob/master/image/electricalcircuit.png
)

![](https://github.com/freyakniglty/SegmentationNotes/blob/master/image/pixelimage.png
)

解带有边界问题的Dirichlet 问题的方程式是一个调和（harmonic）方程。

文章的另一部分介绍了这个方法的一些性质：

- 每一个分割都是与相同标签的种子点连通的。也就是说，没有不包含种子点的孤立的区域。
- 在每一个像素点的概率的K-tuple 和邻界像素点的加权平均K-tuples 等同。权重是由walker biases得到的。
- 解是唯一的。
- 由独立的相同意义的随机变量得到的纯噪音图片的期望分割，是与中性（neutral）分割相同的。（neutral定义：如果图片是neutral的，分割也是neutral。比如缺失意义的内容）



## 算法实现

### 定义与设置

1.定义边的权重：$w_{i,j} = \exp(-\beta(g_i-g_j)^2)$

2.Dirichlet problem

连续型（general case):$D[u] = \frac{1}{2}\int_\Omega|\nabla u|^2d\Omega$

关键在于找到调和方程 u 满足拉普拉斯等式 $\nabla^2u = 0$

3.定义拉普拉斯矩阵：

$L_{ij} = \begin{cases}d_i&&if \ i=j,\\-\omega_{ij} &&if\ v_i\  and\ v_j\ are\ adjacent\ nodes,\\0&&otherwise,\end{cases}$

4.定义关联矩阵：

$A_{e_{ij}v_k} = \begin{cases}+1&&if\ i=k,\\-1&&if\ j=k,\\0&&otherwise,\end{cases}$

把A解释为gradient operator(梯度算子)，$A^T$ 理解为散度。

5.定义C为一个构造矩阵。意义是对角线上每一条边的权重。



### 算法核心

$D[x] = \frac{1}{2}(Ax)^TC(Ax) = \frac{1}{2}x^TLx = \frac{1}{2}\sum_{e{ij}\in E} w_{ij}(x_i-x_j)^2$

求D的最小值。

标记M,U分别为已标记的种子点和未标记的非种子点，则有

$D[x_U] = \frac{1}{2}[x_M^Tx_U^T]\begin{bmatrix}L_M&B\\B^T&L_U\end{bmatrix}\begin{bmatrix}x_M\\x_U\end{bmatrix}=\\ \frac{1}{2}(x_M^TL_Mx_M+2x_U^TB^Tx_M+x_U^TL_Ux_U)$

求一阶导得到：$\boxed{L_Ux_U=-B^Tx_M}$

这个是线性方程组。

设$x^s_i$ 为点$v_i$ 的标签s的概率。则对于种子点$ m_j^s=\begin{cases}1&if\ s\\0 &if\ not \ s\end{cases}$

所以线性方程组又能写成$\boxed{L_Ux^s=-B^Tm^s}$  对一个标签s来说

或者写成$\boxed{L_UX=-B^TM}$ 对所有标签来说。

而且有：$\sum_s x_i^s = 1,\forall v_i\in V$   

#### 步骤总结

1.计算网格上的边的权重

2.得到种子点（交互的或者自动的方式）

3.代入方程组求解。并附带条件$\sum_s x_i^s = 1,\forall v_i\in V$  

4.选择{$x^s_i$} 的最大值，得到最后分割结果。





## 一些性质在数学上的证明

（略）原文更详细。



## random walker的一些行为性质

A.弱边界

B.噪声鲁棒性

C.模糊的未标记区域（不需要很强的直觉上的分界）（ambiguous unseeded regions）







