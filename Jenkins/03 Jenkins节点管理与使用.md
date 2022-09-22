---
title: 'Jenkins节点管理与使用'
tags:
  - Jenkins
categories:
  - Jenkins
date: 2022-09-01 13:51:00
top_img: transparent
cover: https://img1.baidu.com/it/u=1706157547,2217937168&fm=253&fmt=auto&app=138&f=JPG?w=889&h=500
---

### 前提

> Slave节点需要安装java环境(推荐java11)
>
> Centos 安装Java多种方式：https://blog.csdn.net/Xx47585874/article/details/124849471

![image-20220902131433607](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902131433607.png)

### Master/Slave模式

对于大型构件项目而已，一台机器肯定是不够用的，Jenkins支持Master/Slave模式，可以添加任意多的从节点，将复杂任务分发出去。

另外对于不同的构建环境要求，比如Linux环境和Windows环境需要不同节点构建和测试，也需要设置多个从节点。

### 一、找到节点管理的位置

![image-20220901135423222](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901135423222.png)

**节点管理界面详解：**

![image-20220901135550291](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901135550291.png)

### 二、添加节点（添加主机）

![image-20220901135611219](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901135611219.png)

**点击创建后，会让你开始填写节点的必要信息**

![image-20220901135730905](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901135730905.png)

**SSH连接必要信息填写**
下拉选择框按照图示选择即可

![image-20220901135750988](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901135750988.png)

**添加凭据（账号密码）**

![image-20220901135814057](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901135814057.png)

**使用凭据**

![image-20220901135829399](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901135829399.png)

完成上述步骤后，点击保存，就完成了节点的创建

### 三、激活节点

![image-20220901135848794](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901135848794.png)

**此时需要连接刚刚创建的节点**

![image-20220901135943086](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901135943086.png)

**开始连接节点**

![image-20220901140117145](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901140117145.png)

![image-20220901140141154](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901140141154.png)

**再回到节点管理列表刷新查看结果**

![image-20220901140209589](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901140209589.png)

