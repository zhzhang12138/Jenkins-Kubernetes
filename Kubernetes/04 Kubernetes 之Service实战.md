---
title: 'Kubernetes 之Service实战'
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-09-02 13:32:00
top_img: transparent
cover: https://img2.baidu.com/it/u=2608987192,3553019956&fm=253&fmt=auto&app=138&f=JPEG?w=889&h=500
---

### 什么是 [Service](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/ )

> 将运行在一组 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 上的应用程序公开为网络服务的抽象方法。
>
> 使用 Kubernetes，你无需修改应用程序即可使用不熟悉的服务发现机制。 Kubernetes 为 Pod 提供自己的 IP 地址，并为一组 Pod 提供相同的 DNS 名， 并且可以在它们之间进行负载均衡。

### [ 动机](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#动机)

> 创建和销毁 Kubernetes [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 以匹配集群的期望状态。 Pod 是非永久性资源。 如果你使用 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 来运行你的应用程序，则它可以动态创建和销毁 Pod。
>
> 每个 Pod 都有自己的 IP 地址，但是在 Deployment 中，在同一时刻运行的 Pod 集合可能与稍后运行该应用程序的 Pod 集合不同。
>
> 这导致了一个问题： 如果一组 Pod（称为“后端”）为集群内的其他 Pod（称为“前端”）提供功能， 那么前端如何找出并跟踪要连接的 IP 地址，以便前端可以使用提供工作负载的后端部分？
>
> 通过Service

### **创建并启动Service**

> 创建Service

> nginx-service.yaml 

```yaml
apiVersion: v1                  # 指定的api版本，要符合kubectl apiVersion规定，v1是稳定版，必选参数
kind: Service                   # k8s资源类型，采用Service部署，必选参数
metadata:                       # 资源的元数据语句块，是针对kind对应资源的全局属性，必选参数
  name: nginx-service    				# 自定义名称，Service资源名字
  namespace: default            # 默认命名空间
spec:                           # 规格语句块
  ports:                        # 端口配置语句块
    - port: 81                	# 本Service的端口，可以自定义
      targetPort: 80          	# 目标端口号，也就是根据selector基于标签app: nginx选中的pod里容器的端口
      nodePort: 31113           # 自定义对外暴露端口
  selector:                     # 选择器，基于标签进行选择
    app: nginx           				# 标签对，指向pod，key值为nginx
  type: NodePort                # service模式，有ClusterIP,NodePort,Loabalancer等
```

> 启动Service
>
> kubectl apply -f nginx-service.yaml 

> kubectl get service													// 获取Service
>
> kubectl delete svc [service名] -n default				// 删除Service

![image-20220902155656126](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902155656126.png)

### **验证结果**

> http://121.4.213.210:31113/

![image-20220902155447982](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902155447982.png)

### 参考网站

> K8S系列学习之SERVICE实战
>
> https://www.freesion.com/article/37841433636/





















