--- 
layout: post
title: LDA的简单实现方法(草稿中)
---


LDA(Latent Dirichlet allocation)是一个非常不错的主题模型模型。它是在PLSA的基础上再加上两个Dirichlet分布的超参alpha和beta，使得结果更加可靠。LDA是一个无监督训练模型，训练完成后可以取得文档集中文档的主题分布，通常情况下训练时只需要指定一个参数－－即主题的数量k即可，简单方便。

LDA最早时候由Blei、Ng等人在2003年提出，现在应用范围也比较广泛，包括文本分类和聚类、相似度计算、还有query优化等等。具体应用可以去看这里[LDA(latent dirichlet allocation)的应用](http://www.zhizhihu.com/html/y2013/4219.html)。

原理在这里就不多解释了，因为它所使用到的知识非常杂，包括首先需要去了解PLSA和EM算法，然后是概率论知识、Gibbs取样、多项式-Dirchlet共轭等等，最后把这些全部结合在一起才能够最终推导出我们需要使用的公式（当时机器学习课上老师花了将近一个月的时间才教完）。如果喜欢去钻研的，这里推荐一下richjin的[LDA数学八卦][1]这个对于想自学Gibbs Sampling LDA的话非常不错。

其实最早的时候，Blei的LDA论文中使用的是EM算法，不过这个EM算法的LDA有点复杂，所以单纯想实现LDA的话去参考原始论文是没有多大价值的。目前比较流行的则是通过Gibbs Sampling的方法去估计LDA中各个参数的值，非常简单。可以去参考一下这篇Technic Report [Parameter estimation for text analysis](www.arbylon.net/publications/text-est.pdf‎)，LDA实现的代码说明中大多都引用了这篇。

虽然说推导的过程非常复杂，但是LDA最终的结果却非常简单，这个也就是好的数学模型所具有的特点。虽然是实现，但是稍微说一下LDA的基本知识还是必要的。

![LDA概率图模型](LDA-Graph.png)

以上这张就是LDA的概率图模型，LDA假设一篇文章由多个主题组成（图中的`\( \overrightarrow{\theta_m} \)`），文章中每一个词均对应这篇文中其中一个主题（`\( z_{m, n} \)`）就代表文章m中第n个词对应的主题）。因此LDA模型生成一篇文章的过程是：

* 首先从`\( \alpha \)`为参数的Dirchlet分布中取样生成一个多项式分布`\( \overrightarrow{\theta_m} \)`，这个就是文档m的主题分布。
* 然后从多项式分布`\( \overrightarrow{\theta_m} \)`中取样生成文档m的第n个词的主题`\( z_{m, n} \)`
* 接着从`\( \beta \)`为参数的Dirchlet分布中取样生成主题`\( k = z_{m, n} \)`对应的词的多项式分布`\( \overrightarrow{\phi_m} \)`
* 最后从多项式分布`\( \overrightarrow{\phi_m} \)`中随机取样一个词语，生成文章m的第n个词。

这个过程看上去有点难懂，这里只需要简单的知道每个符号大致代表的意义即可。其中我们最重要求的是`\( \overrightarrow{\theta} \)`和`\( \overrightarrow{\phi} \)`。`\( \overrightarrow{\theta} \)`就是每篇文章的主题分布矩阵，也就是我们要求的最终结果，用这个可以知道每篇文章属于哪几个主题或者可以用KL距离去计算文章之间的相关程度。`\( \overrightarrow{\phi} \)`就是每个主题对应的词的分布，使用他可以计算一篇新文章的主题分布，也可以用来计算词和词之间的关系。

在[LDA数学八卦][1]中详细介绍了每次Sampling中一个词属于某主题概率公式的推导过程，这个公式也就是整个Gibbs Sampling LDA的核心部分。这个公式就是



[1]: http://www.52nlp.cn/lda-math-%E6%B1%87%E6%80%BB-lda%E6%95%B0%E5%AD%A6%E5%85%AB%E5%8D%A6 
[2]: www.arbylon.net/publications/text-est.pdf‎