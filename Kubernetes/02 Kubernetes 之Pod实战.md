---
title: 'Kubernetes 之Pod实战'
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-09-02 16:11:00
top_img: transparent
cover: https://img2.baidu.com/it/u=2608987192,3553019956&fm=253&fmt=auto&app=138&f=JPEG?w=889&h=500
---

### 什么是 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/ )

> **Pod** 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。
>
> **Pod**（就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） [容器](https://kubernetes.io/zh-cn/docs/concepts/overview/what-is-kubernetes/#why-containers)； 这些容器共享存储、网络、以及怎样运行这些容器的声明。 Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。 Pod 所建模的是特定于应用的 “逻辑主机”，其中包含一个或多个应用容器， 这些容器相对紧密地耦合在一起。 在非云环境中，在相同的物理机或虚拟机上运行的应用类似于在同一逻辑主机上运行的云应用。

### **编写Pod的yaml文件**

> pod1.yaml

```yaml
apiVersion: v1												# 指定的api版本，要符合kubectl apiVersion规定，v1是稳定版本，必选参数
kind: Pod															# k8s资源类型，常见的有Pod、Deployment、Service、RS、RC等待，必选参数
metadata:															# 资源的元数据语句块，对应资源的全局属性，必选参数
  name: pod1													# 该资源的名字，可以自定义，在同一个namesoace中必须唯一	
  namespace: default									# 指定该资源的命名空间，默认是default
  containers:													# 容器语句块，设定具体容器相关参数，比如名称，镜像，端口等等
  - name: testpod-dev									# 容器名称，可以自定义，建议可读性
    image: nginx											# 容器镜像，这里设置的是k8s公共镜像仓库中的nginx，可设置版本，默认是最新的
    ports:														# 该容器的端口设置语句块
    - containerPort: 80								# 该容器的端口，80
```

### **提交yaml文件并创建Pod**

> 在这个小实验里，只创建了一个Pod，且该Pod中只有一个容器，该容器安装的是nginx最新版本的镜像，也就是部署了一个nginx容器。

> kubectl apply -f pod1.yaml
>
> kubectl get pod

![image-20220902163758989](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902163758989.png)

### **验证结果**

> 可以使用kubectl命令行直接进入Pod内，然后查看nginx安装文件。如果想使用curl命令看nginx欢迎页面信息，单独操作Pod是不行的，需要配合Service组件来使用，后续会单独分享Service。毕竟k8s是集群调度，无论是从物理层面的“垂直维度”，还是从编排层面的“水平维度”，都是做了隔离的，这跟单独操作虚拟机还是有较大区别的。

> kubectl exec -it pod1 -c testpod-dev -- /bin/bash   
>
>  //pod1是本实验的Pod名称，testpod-dev是本实验pod1中的容器名称，该docker容器部署了nginx

![image-20220902164009395](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902164009395.png)

> 以上红色方框中的配置文件，就是咱们熟悉的nginx配置文件了，/conf.d/default.conf中可以修改监控端口（默认是80），nginx.conf中可以修改worker_processes的数量。

### **一个Pod中部署多个docker容器**

> 这里就开始体现k8s的好处了，只需要在原来的pod1.yaml文件上里的spec语句块中，继续增加容器规格参数即可。为了方便对比，增加了代码的文件叫pod-1.yaml，同时名称定义为pod-1。

> pod-1.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: default
spec:
  containers:
  - name: testpod-dev
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  - name: testpod-dev-2
    image: ccr.ccs.tencentyun.com/flask_script/flask_script:v1
    ports:
    - containerPort: 81
```

然后使用同样的命令，创建pod-1

> kubectl apply -f pod-1.yaml         //增加了my-tomcat容器的yaml文件
>
> kubectl get pods        					//查看pod状态，直到状态变成Running

![image-20220902165914258](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902165914258.png)

> 做个对比，pod1中只有一个docker容器，而pod-1中有两个docker容器。

>  kubectl describe pod pod1

![image-20220902170121982](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902170121982.png)

> kubectl describe pod pod-1

![image-20220902170243428](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902170243428.png)

### **在Pod中限制docker容器的资源使用**

> 为了好做对比，在pod1.yaml基础上复制一个pod-11.yaml文件，然后增加资源设置语句块。

>  pod-11.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-11
  namespace: default
spec:
  containers:
  - name: testpod-dev
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  resources:
    requests:
      cpu: 0.5
      memory: 500Mi
    limits:
      cpu:1
      memory:800Mi
```

然后让pod1（没有设置资源限制）和pod-11（设置了资源限制）都Running起来，然后使用命令查看当前Node上的运行情况。

> 运行 kubectl apply -f pod-11.yaml

![image-20220902170927476](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902170927476.png)

> kubectl get pod -o wide    							//查看pod在哪些Node上

![image-20220902171120530](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902171120530.png)

> kubectl describe node k8s-node01  		 //查看k8s-node2这个Node上的运行信息

![image-20220902171211146](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902171211146.png)

### 更多配置项及解释可参照如下

> 转自网络：https://www.jianshu.com/p/35dde2b1951b ）

```yaml
apiVersion: v1 #指定api版本，此值必须在kubectl apiversion中  
kind: Pod #指定创建资源的角色/类型  
metadata: #资源的元数据/属性  
  name: web04-pod #资源的名字，在同一个namespace中必须唯一  
  labels: #设定资源的标签，详情请见http://blog.csdn.net/liyingke112/article/details/77482384
    k8s-app: apache  
    version: v1  
    kubernetes.io/cluster-service: "true"  
  annotations:            #自定义注解列表  
    - name: String        #自定义注解名字  
spec:#specification of the resource content 指定该资源的内容  
  restartPolicy: Always #表明该容器一直运行，默认k8s的策略，在此容器退出后，会立即创建一个相同的容器  
  nodeSelector:     #节点选择，先给主机打标签kubectl label nodes kube-node1 zone=node1  
    zone: node1  
  containers:  
  - name: web04-pod #容器的名字  
    image: web:apache #容器使用的镜像地址  
    imagePullPolicy: Never #三个选择Always、Never、IfNotPresent，每次启动时检查和更新（从registery）images的策略，
                           # Always，每次都检查
                           # Never，每次都不检查（不管本地是否有）
                           # IfNotPresent，如果本地有就不检查，如果没有就拉取
    command: ['sh'] #启动容器的运行命令，将覆盖容器中的Entrypoint,对应Dockefile中的ENTRYPOINT  
    args: ["$(str)"] #启动容器的命令参数，对应Dockerfile中CMD参数  
    env: #指定容器中的环境变量  
    - name: str #变量的名字  
      value: "/etc/run.sh" #变量的值  
    resources: #资源管理，请求请见http://blog.csdn.net/liyingke112/article/details/77452630
      requests: #容器运行时，最低资源需求，也就是说最少需要多少资源容器才能正常运行  
        cpu: 0.1 #CPU资源（核数），两种方式，浮点数或者是整数+m，0.1=100m，最少值为0.001核（1m）
        memory: 32Mi #内存使用量  
      limits: #资源限制  
        cpu: 0.5  
        memory: 32Mi  
    ports:  
    - containerPort: 80 #容器开发对外的端口
      name: httpd  #名称
      protocol: TCP  
    livenessProbe: #pod内容器健康检查的设置，详情请见http://blog.csdn.net/liyingke112/article/details/77531584
      httpGet: #通过httpget检查健康，返回200-399之间，则认为容器正常  
        path: / #URI地址  
        port: 80  
        #host: 127.0.0.1 #主机地址  
        scheme: HTTP  
      initialDelaySeconds: 180 #表明第一次检测在容器启动后多长时间后开始  
      timeoutSeconds: 5 #检测的超时时间  
      periodSeconds: 15  #检查间隔时间  
      #也可以用这种方法  
      #exec: 执行命令的方法进行监测，如果其退出码不为0，则认为容器正常  
      #  command:  
      #    - cat  
      #    - /tmp/health  
      #也可以用这种方法  
      #tcpSocket: //通过tcpSocket检查健康   
      #  port: number   
    lifecycle: #生命周期管理  
      postStart: #容器运行之前运行的任务  
        exec:  
          command:  
            - 'sh'  
            - 'yum upgrade -y'  
      preStop:#容器关闭之前运行的任务  
        exec:  
          command: ['service httpd stop']  
    volumeMounts:  #详情请见http://blog.csdn.net/liyingke112/article/details/76577520
    - name: volume #挂载设备的名字，与volumes[*].name 需要对应    
      mountPath: /data #挂载到容器的某个路径下  
      readOnly: True  
  volumes: #定义一组挂载设备  
  - name: volume #定义一个挂载设备的名字  
    #meptyDir: {}  
    hostPath:  
      path: /opt #挂载设备类型为hostPath，路径为宿主机下的/opt,这里设备类型支持很多种  
```

