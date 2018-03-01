# 简介

本文分割方法主要是minima rule，就是判断负曲率的最小值 或者说是开始变凸（concave）。
另一方面可以用sharp edges(凹或者凸)，甚至是平滑的边分割，取决于不同的标准。

random walks的核心思想是：先选择一个集合的种子点（用户输入）。对于每一个非种子点，决定它到每一个种子点的概率，这个由它到每个邻域的概率决定。当每个非种子点被标记上不同种子点的标签后，分割完成。

对于粗糙的分割，只需要很少的种子点，并且它们越分散越好，结果是只能分割成几个区域。
对于精细的分割，种子点是根据特征敏感的各向同性点采样（feature sensitive isotropic point sampling 专有的统计学名词有论文介绍）方法自动选取的。种子数远远多于需要的区域数，最后根据相似度进行区域融合。

# 相关

对于不同目的，可以把分割分成两类。第一类是CAD的工业图，有标准的几何体。另外一类是自然界存在的物体，比如说人体骨骼等等。

# 交互分割

主要方法与步骤：假设用户选取了n个面作为种子点（面）。n是最终需要的分割区域个数。因此种子必须是撒在需要的区域中（也就是不能让两个种子点撒在同一个区域中）。设种子点(面)为<img src="http://latex.codecogs.com/gif.latex?s_1,s_2....s_n"/>。非种子面为<img src="http://latex.codecogs.com/gif.latex?f_1,f_2.....f_m"/>。因为是三角网格，对于任意一个非种子面<img src="http://latex.codecogs.com/gif.latex?f_k"/>，有三条边<img src="http://latex.codecogs.com/gif.latex?e_{k,1},e_{k,2},e_{k,3}"/> 对应了三个概率<img src="http://latex.codecogs.com/gif.latex?p_{k,1},p_{k,2},p_{k,3}"/>。这三个概率就是f_k到邻域（三个邻面）的random walk 行为。哪个概率高就去哪个邻面。
满足以下条件<br />
![](http://latex.codecogs.com/gif.latex?{\sum_{i=1}^3{p_{k,i}}}=1)  

记<img src="http://latex.codecogs.com/gif.latex?f_k"/> 到 <img src="http://latex.codecogs.com/gif.latex?s_l"/> 的概率(在先到达其他种子点之前)为 <img src="http://latex.codecogs.com/gif.latex?P^l(f_k)"/>，则有：<img src="http://latex.codecogs.com/gif.latex?P^l(s_l) = 1"/> 和 <img src="http://latex.codecogs.com/gif.latex?P^l(s_k)=0"/> 对任意 k ≠ l.
对一个非种子点则有：<br />
![](http://latex.codecogs.com/gif.latex?{{P}^l{(f_k)}}=\sum_{i=1}^3p_{k,i}P^l(f_{k,i}))  


因为有m个非种子点，则可以写成![](http://latex.codecogs.com/gif.latex?{A_{m\times{m}}}{P^l}={B^l}) 

对与种子点相邻的非种子点，<img src="http://latex.codecogs.com/gif.latex?P^l(f_k)"/> 要么是0，要么是1(如果种子点是<img src="http://latex.codecogs.com/gif.latex?s_l"/> 则为1，不是<img src="http://latex.codecogs.com/gif.latex?s_l"/>则为0)。因为种子点最多有三个邻面，所以<img src="http://latex.codecogs.com/gif.latex?B^l"/>最多有三个1。
因为有n个种子点，所以得到<img src="http://latex.codecogs.com/gif.latex?P_{mxn}"/>即有n column的矩阵。也有<img src="http://latex.codecogs.com/gif.latex?B = (B_1，...,B_n)"/>

所以整个线性方程组为  AP=B

这是个稀疏线性方程组。

解出来的P后，根据
![](http://latex.codecogs.com/gif.latex?{{P}^l{(f_k)}}=\max_{t=1,...,n}P^t(f_k)) 

 
得到非种子点的标签l。也就是一个row里面取最大值。最后得到一个向量。再把对应的P找到对应的l，得到一个m维的标签向量。

## 概率计算
### 图形模型
![](http://latex.codecogs.com/gif.latex?{d_1(f_i,f_{i,k})}=\eta{\[dihedral(f_i,f_{i,k})\]} = {\frac{\eta}{2}}{{\|\|N_i-N_{i,k}\|\|}^2}) 
![](https://github.com/freyakniglty/SegmentationNotes/blob/master/image/d.png)

dihedral angle有自带函数可以获取。η = 1.0 或者 η=0.2 取决于是凸的还是凹的。有简易的判断方法。
![](http://latex.codecogs.com/gif.latex?d(f_i,f_{i,k})=\frac{d_1(f_i,f_{i,k})}{\overline{d_1}})  

 
<img src="http://latex.codecogs.com/gif.latex?\overline{d_1}"/>是均值。
最后，概率公式为：
![](http://latex.codecogs.com/gif.latex?p_{i,k}=\|{e_{i,k}}\|\exp(-\frac{d(f_i,f_{i,k})}{\sigma}))  


σ = 1.0 可以符合大多数情况。

### 工业模型（省略）

## 一些前处理和后处理

对区域间的边界进行光滑处理。

## 自动分割

可以选择比所需区域个数更多的种子点。

多于图形模型，经常只需要粗略分割，不需要非常多的种子点的集合，但是对于工业模型需要细节上的分割，可能需要更多的种子点。

也可以允许用户选择特殊的种子点，在自动选取种子点之前或者之后放进去。

最后需要进行区域融合，将过度分割的区域融合成数目和分割合理的区域。

### 粗略种子选取

使用与k-means相似的聚合方式选择种子点。

$D(f_i,f_j)$ 是$f_i$ 到 $f_j$ 的最短路径。

$s_1,s_2,...,s_n$ 是已经选好的种子点。

$s_{n+1}=  \arg\max_{f_k\in F}\{\min_{i=1,...,n} D(f_k,s_i)\}$

以上公式表面了如何选择第n+1个种子点。 

当$s_{n+1}$ 到最近的种子点的D显著减小时，种子选取结束。

### 精细种子选取

![](https://github.com/freyakniglty/SegmentationNotes/blob/master/image/fine%20selecting.png)

#### 自动选取

[Feature sensitive sampling](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.21.876)

$O(n log m)$ 为计算度

#### 融合

分割完成后，有相邻的两个区域$S_i,S_j$

标记，$\partial S_i\cap\partial S_j$ 为两个区域的共同边界。$\partial S_i\cup\partial S_j$ 为两个区域的联合边界。

标记，$D_{\partial S_i\cap\partial S_j} = \sum_{e\in\partial S_i\cap\partial S_j}|e|d_e$ ，

其中|e| 是 边的长度，$d_e$ 是两个与e相邻的面的距离。

定义$L_{\partial S_i\cap\partial S_j} = \sum_{e\in\partial S_i\cap\partial S_j}|e|$ 为共同边的总长度。

$，D_{\partial S_i\cap\partial S_j}，L_{\partial S_i\cap\partial S_j}$ 

然后定义融合消耗 $c_{i,j}$ 为：

$c_{i,j} = \frac {D_{\partial S_i\cap\partial S_j}/L_{\partial S_i\cap\partial S_j}}{D_{\partial S_i\cup\partial S_j}/L_{\partial S_i\cup\partial S_j}}$  

选择最小的$c_{i,j}$ 将其中的两个区域融合，然后更新新的{$c_{i,j}$} 向量。当最小的$c_{i,j}$ 大约为0.5的时候融合终止。








