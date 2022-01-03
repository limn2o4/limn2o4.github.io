---
layout:     post
title:      "BigDL 实战到原理（二） BigDL训练原理"
subtitle:   " \"BigDL训练原理\""
date:       2022-1-2 12:00:00
author:     "limn2o4"
catalog: false
tags:
    - spark
    - tensorflow
    - bigdl
    - 分布式训练
    - 分布式系统
---
想要解决集群上的深度学习方案，分布式训练则是头等大事。分布式方案比较成熟的是horovod，实现的是百度的ring-allreduce，tensorflow自带的是简单的allreduce，二者在网络传输效率下，horovod更胜一筹。BigDL则借助Spark机制，巧妙的设计了一种通信量介于二者之间的方式。下文我们分析下BigDL分布式训练的流程
## 前言1 分布式训练之数据并行
分布式训练的方式大致可分三种
- 数据并行：将大量的数据分片，每个节点负责一部分数据的训练
- 模型切分：将参数比较多的模型层切分到不同结构的机器上，常见的有将embedding层横向切分，反正每个参数都是单独更新
- 流水线并行：可以将执行逻辑并行起来，将计算和通信重叠  
数据并行最简单的实现就是allreduce，下面是他的运行流程