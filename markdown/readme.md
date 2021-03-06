# Spark Internals

Spark Version: 1.0.2  
Doc Version: 1.0.2.0

## Authors
| Weibo Id | Name | 
|:-----------|:-------------|
|[@JerryLead](http://weibo.com/jerrylead) | Lijie Xu | 

## Introduction

本文主要讨论 Apache Spark 的设计与实现，重点关注其设计思想、运行原理、实现架构及性能调优，附带讨论与 Hadoop MapReduce 在设计与实现上的区别。不喜欢将该文档称之为“源码分析”，因为本文的主要目的不是去解读实现代码，而是尽量有逻辑地，从设计与实现原理的角度，来理解 job 从产生到执行完成的整个过程，进而去理解整个系统。

讨论系统的设计与实现有很多方法，本文选择 **问题驱动** 的方式，一开始引入问题，然后分问题逐步深入。从一个典型的 job 例子入手，逐渐讨论 job 生成及执行过程中所需要的系统功能支持，然后有选择地深入讨论一些功能模块的设计原理与实现方式。也许这样的方式比一开始就分模块讨论更有主线。

本文档面向的是希望对 Spark 设计与实现机制，以及大数据分布式处理框架深入了解的 Geeks。

因为 Spark 社区很活跃，更新速度很快，本文档也会尽量保持同步，文档号的命名与 Spark 版本一致，只是多了一位，最后一位表示文档的版本号。

由于技术水平、实验条件、经验等限制，当前只讨论 Spark core standalone 版本中的核心功能，而不是全部功能。诚邀各位小伙伴们加入进来，丰富和完善文档。

关于学术方面的一些讨论可以参阅相关的论文以及 Matei 的博士论文，也可以看看我之前写的这篇 [blog](http://www.cnblogs.com/jerrylead/archive/2013/04/27/Spark.html)。

好久没有写这么完整的文档了，上次写还是三年前在学 Ng 的 ML 课程的时候，当年好有激情啊。这次的撰写花了 20+ days，从暑假写到现在，大部分时间花在 debug、画图和琢磨怎么写上，希望文档能对大家和自己都有所帮助。


## Contents
本文档首先讨论 job 如何生成，然后讨论怎么执行，最后讨论系统相关的功能特性。具体内容如下：

1. [Overview](https://github.com/JerryLead/SparkInternals/blob/master/markdown/1-Overview.md) 总体介绍
2. [Job logical plan](https://github.com/JerryLead/SparkInternals/blob/master/markdown/2-JobLogicalPlan.md) 介绍 job 的逻辑执行图（数据依赖图）
3. [Job physical plan](https://github.com/JerryLead/SparkInternals/blob/master/markdown/3-JobPhysicalPlan.md) 介绍 job 的物理执行图
4. [Shuffle details](https://github.com/JerryLead/SparkInternals/blob/master/markdown/4-shuffleDetails.md) 介绍 shuffle 过程
5. [Architecture](https://github.com/JerryLead/SparkInternals/blob/master/markdown/5-Architecture.md) 介绍系统模块如何协调完成整个 job 的执行
6. [Cache and Checkpoint](https://github.com/JerryLead/SparkInternals/blob/master/markdown/6-CacheAndCheckpoint.md)  介绍 cache 和 checkpoint 功能
7. [Broadcast](https://github.com/JerryLead/SparkInternals/blob/master/markdown/7-Broadcast.md) 介绍 broadcast 功能
8. Job Scheduling 尚未撰写
9. Fault-tolerance 尚未撰写

可以直接点 md 文件查看。

如果使用 Mac OS X 的话，推荐下载 [MacDown](http://macdown.uranusjr.com/) 后使用 github 主题去阅读这些文档。

## Examples
写文档期间为了 debug 系统，自己设计了一些 examples，放在了 [SparkLearning/src/internals](https://github.com/JerryLead/SparkLearning/tree/master/src/internals) 下。

## Acknowledgement

文档写作过程中，遇到过一些细节问题，感谢下列同学给予的解答和帮助：

[@Andrew-Xia](http://weibo.com/u/1410938285)

[@CrazyJVM](http://weibo.com/476691290)

[@王联辉](http://weibo.com/u/1685831233)

[@Joshuawangzj](http://weibo.com/u/1619689670)

特别感谢 [@明风Andy](http://weibo.com/mingfengandy) 同学给予的大力支持。