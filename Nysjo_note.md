# 简介

根据骨头密度变化和图像误差比如噪声和部分体影响（partial volume effects）,在ct图像中相邻的骨头和骨头部分（bone fragments）可以按类型按阈值相连。
人工的方法是用软件itk-snap。但非常耗时。

现在可以利用random walks的算法，然后用3D 编辑软件修改结果。

## 相关


	* 以模型为基础的分割方法可以用于自动分割，但是不够一般化。

	* 人工分割可以提供精确的结果，但太耗时。

	* 半自动或者交互的分割方法结合了不精确的用户输入和合适的算法。



# 方法

## 步骤

	1. 整体性骨骼分割 collective segmentation

	2. 标记个体的骨头结构

	3. random walk 分离

	4. 分割编辑


## 整体性骨骼分割

根据灰度的阈值分割。这个阈值可以人工设定。阈值预设为300HU（HU是量化射线下透明度的单位），或者选择HU的下界，比如疏松的骨头。带有噪声的图像可以用高斯过滤（σ=0.6）来光滑化。可以多次使用高斯滤波器，但一般一次就够了。这个过程可利用多线程。

## 等值面延迟渲染

使用GPU加速的raycasting算法来把骨头渲染成等值面。等值面的值设定成t_bone, 这样可以可视化阈值分割。
通过多渲染目标技术（MRT）在G-buffer (几何缓冲区)中渲染 first-hit 位置，表面的法向信息，然后计算阴影和局部光照在一个附加的通道。分割标签被储存在一个分离的3D纹理中，然后被在光照通道中的最近的sampling取得。

局部光照由带有法向信息的Blinn-Phong 渲染模型计算出。
结合局部光照和阴影。阴影由first-hit纹理（通过单个方向光线的视点渲染的）得出。PCF 和Poisson disc sampling过滤。
分割时可以关闭阴影如果它们模糊了细节。

散射光（ambient lighting），可以选择基于图像的多颜色强度的。来可视化分割和区分金属。从local ambient occlusion模型得到，用Monte-Carlo integration来建立。


## 3D纹理的交互界面

比起2D切片有可视化的优势。

为了避免画笔渗入小的缝隙，可以用first-hit 纹理实现种子挑选，计算local ambient occlusion，然后丢掉画笔在ambient occlusion value在画笔中心超过一定阈值的绘制。

附加的工具是，label选择笔，橡皮擦，填充工具，编辑工具（见2.6）。裁剪工具。


上图。交互的基本流程。

## Random Walks

在整体性分割之后，第二步就是random walks 分割。

random walks 鲁棒性，弱边界性，可以用多标签，没有小分割的缺陷。但是消耗计算和内存。

核心思想是，对于每一个体元选择最大的那个概率的标签。vertices 代表每个体元，edges代表每个体元对6个邻域的连接情况。eij 被权重 基于梯度的 wij代表。 
gi,gj是vi,vj上的灰度intensity, β 是一个变量用于决定梯度（gradient magnitude） 的影响。ε =0.01 常量。
增加β 可使 random walkers 更少地倾向于横跨有高梯度的边。 我们一般设置 β = 3000 对于骨分割。2000-4000 都可以。

利用稀疏线性系统来得到每一个标签的结果。通过第一步得到的整体骨头分离图像（collective segmentation）而不是整个图像。

为了避免稀疏矩阵A 成为奇异矩阵（因为有些骨头的部分是孤立的，没有与整块相接）。给对角线上的元素加一个k = 0.001 。

## Iterative Solver
 
Jacobi preconditioned conjugate gradient (CG) 

这个部分CPU太慢。最好用GPU实现。

## 分割编辑

可以进行多次运行solver和增加种子点对于第一次random walks的结果。
volume clipping : 暂时将分割出去的部分灰度值设为t_bone - 1，再更新模型的灰度。

动态的编辑工具。实时动态增加标签，增长对比。







