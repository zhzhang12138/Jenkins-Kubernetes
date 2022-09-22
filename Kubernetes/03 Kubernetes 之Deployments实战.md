---
title: 'Kubernetes 之Deployments实战'
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-09-02 13:32:00
top_img: transparent
cover: https://img2.baidu.com/it/u=2608987192,3553019956&fm=253&fmt=auto&app=138&f=JPEG?w=889&h=500
---

### 什么是 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/ )

> 一个 Deployment 为 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 和 [ReplicaSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/) 提供声明式的更新能力。
>
> 你负责描述 Deployment 中的 **目标状态**，而 Deployment [控制器（Controller）](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/) 以受控速率更改实际状态， 使其变为期望状态。你可以定义 Deployment 以创建新的 ReplicaSet，或删除现有 Deployment， 并通过新的 Deployment 收养其资源。

### 创建 Deployment

下面是一个 Deployment 示例。其中创建了一个 ReplicaSet，负责启动三个 `nginx` Pods：

> nginx-deployment.yaml

```yaml
apiVersion: apps/v1								 # 指定的api版本，要符合kubectl apiVersion规定，v1是稳定版，必选参数
kind: Deployment									 # k8s资源类型，采用Deployment部署，必选参数
metadata:													 # 资源的元数据语句块，是针对kind对应资源的全局属性，必选参数
  name: nginx-deployment					 # 自定义名称，基于此yaml文件创建的pod，都带有此处名称的前缀
spec:															 # 规格语句块
  replicas: 3											 # 目标副本数，创建的pod维持2个副本，也就是两个pod
  selector:												 # 标签选择器，deployment操作pod，就是通过此功能实现的
    matchLabels:									 # 匹配标签语句块，在此语句块中可以写单个标签，也可以写标签集合
      app: nginx									 # 单个标签，key是app，value是nginx
  template:												 # 创建pod的模版语句块
    metadata:											 # 此模版的元数据语句块
      labels:											 # 基于此模版创建的pod标签组
        app: nginx								 # pod标签，这里的标签需要跟spec.selector.matchLabels中的标签保持一致
    spec:													 # pod模版的规格语句块
      containers:									 # 容器语句块
      - name: nginx								 # 容器名称，可以自定义
        image: nginx:1.14.2				 # 容器镜像地址
        ports:										 # 暴漏容器端口
        - containerPort: 80				 # 定义8080端口
```

通过运行以下命令创建 Deployment 

> kubectl apply -f nginx-deployment.yaml 			//创建deployment		

> kubectl get pod   													 //在默认命名空间下查看pod运行状态
> kubectl describe pod [pod名]   					         //查看具体pod的运行信息
> kubectl get deployment    							         //查看deployment运行结果

![image-20220902151235909](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902151235909.png)

### Deployment升级容器镜像

**通过更新Deployment来升级容器镜像（nginx）**，由1.7.9版本升级到latest版本，并观察Pod的变化

> kubectl set image deployment/nginx-deployment nginx=nginx:latest   //设置nginx最新镜像

![image-20220902151859902](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902151859902.png)

当然也可以指定升级到某个具体的版本。

![image-20220902151916884](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902151916884.png)

### **Deployment回滚机制**

> 如果发现升级的软件版本不稳定，可以很方便的实现回滚。

> kubectl rollout history deployment/nginx-deployment   											  //查看可回滚的历史版本
> kubectl rollout undo deployment/nginx-deployment --to-revision=3						 //回滚到指定历史版本
> kubectl rollout undo deployment/nginx-deployment												    //不指定版本，就是回滚到前一个

![image-20220902152140354](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902152140354.png)

### **Deployment扩容和缩容**

> 这个功能其实跟RS/RC差不多，也是通过控制replicas副本数来实现的。

> kubectl scale deployment nginx-deployment --replicas 5   //扩容到5个副本
> kubectl scale deployment nginx-deployment --replicas 1   //缩容到1个副本

![image-20220902152603891](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902152603891.png)

### 参考网站

> K8S系列学习之DEPLOYMENT实战
>
> https://www.freesion.com/article/97061419584/























