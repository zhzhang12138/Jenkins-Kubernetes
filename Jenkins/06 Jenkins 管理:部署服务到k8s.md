---
title: 'Jenkins管理/部署服务到k8s'
tags:
  - Jenkins
categories:
  - Jenkins
date: 2022-09-01 17:49:00
top_img: transparent
cover: https://img1.baidu.com/it/u=1706157547,2217937168&fm=253&fmt=auto&app=138&f=JPG?w=889&h=500
---

### 前提

> Jenkins需要安装的插件：
>
> **Kubernetes： Jenkins 插件，用于在 Kubernetes 集群中运行动态代理。**
>
> **Kubernetes Cli Plugin：该插件可直接在Jenkins中使用kubernetes命令行进行操作。**
>
> 用此插件可以使用kubectl命令来管理k8s，首先需要在jenkins凭据中添加一个类型为"Secret file" 类型的凭据，上传k8s的config文件
>
> 
>
> 服务器需要安装的工具：
>
> [kubectl](https://kubernetes.io/zh-cn/docs/tasks/tools/)：Kubernetes 命令行工具，kubectl，使得你可以对 Kubernetes 集群运行命令。 你可以使用 kubectl 来部署应用、监测和管理集群资源以及查看日志。

### 获取k8s的config文件

* k8s-master集群

```bash
cat .kube/config

外网通过kubeconfig访问内网下k8s集群
https://zhuanlan.zhihu.com/p/505324148
```

* 腾讯云

![image-20220902092947223](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902092947223.png)

### 添加凭证

![image-20220902093239332](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902093239332.png)

### 设置完后可以用以下类似脚本来管理k8s

```shell
pipeline {
     agent {
        label 'aliyun-node'
    }
    stages {
       stage('Get Pods') {
           steps {
               withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl get pods'
               }
           }
       }
    }
}
```

### 构建结果

![image-20220902093514046](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902093514046.png)
