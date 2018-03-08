# 简介

这篇方法用到全局连续的错误分值，来无监督学习。

两个核心的方法是：comparative reasoning, hashing method。WTA是其中之一（[winner take all](https://en.wikipedia.org/wiki/Winner-take-all_(computing))）

以及Random Walks(RW)。



用WTA进行相似度检索是一个预处理（pre-processing step）

使用DPMMs(Dirichlet Process Mixture Models)的RW方法。

![](https://github.com/freyakniglty/SegmentationNotes/blob/master/image/steps.png)



# [WTA Hash](http://blog.csdn.net/app_12062011/article/details/53462563)



$X_n|X_1,...,X_{n-1} = \begin{cases}X_i&with\ probability\ \frac{1}{n-1+\alpha\\ }\\new\ draw\ from\ G_0& with\ probability\ \frac{\alpha}{n-1+\alpha}\end{cases}$

如果一个新的样本与前面的样本属于同一类则被丢弃，如果一个样本与前面的不同，则作为新的种子点。概率是作为种子点的概率。

