---
title: 'Jenkins pipeline部署k8s应用'
tags:
  - Jenkins
categories:
  - Jenkins
date: 2022-09-01 17:49:00
top_img: transparent
cover: https://img1.baidu.com/it/u=1706157547,2217937168&fm=253&fmt=auto&app=138&f=JPG?w=889&h=500
---

### k8s部署应用流程

*  编写Python代码
* 测试
* 编写 Dockerfile
* 构建打包 Docker 镜像
* 推送 Docker 镜像到仓库
* 编写 Kubernetes YAML 文件
* 利用 kubectl 工具部署应用
* 版本迭代：更改 YAML 文件中 Docker 镜像 TAG
* 通过Jenkins实现自动化部署

### 一、准备应用程序

> 代码链接
>
> https://github.com/zhzhang12138/flask_script.git

![image-20220902100416593](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902100416593.png)

### 二、构建/打包/推送 Docker 镜像

```
# 打包项目(mac m1打包 linux平台)
# docker build -t flask_demo:1.0 -f ./Dockerfile . --platform linux/amd64
# 打包项目(默认平台打包)
docker build -t flask_demo:1.0 -f ./Dockerfile .
# 运行
docker run -d -p 9090:8080 --name flask_demo_01 -t flask_demo:1.0
# * -d: 后台运行容器，并返回容器ID；
# * -p: 指定端口映射，格式为：主机(宿主)端口:容器端口；
# * --name="flask_demo_01": 为容器指定一个名称；
# * -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；



登录腾讯云容器镜像服务 Docker Registry

docker login ccr.ccs.tencentyun.com --username=100018328451复制
从 Registry 拉取镜像

docker pull ccr.ccs.tencentyun.com/flask_script/flask_script:[tag]复制
其中 [tag] 请根据您需要拉取镜像的具体版本镜像替换，如 latest。更多命令说明，请参看官方文档：docker pull

向 Registry 中推送镜像
docker tag [imageId] ccr.ccs.tencentyun.com/flask_script/flask_script:[tag]复制
docker push ccr.ccs.tencentyun.com/flask_script/flask_script:[tag]复制

```

![image-20220902100400173](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902100400173.png)

### 三、编写 Kubernetes YAML 文件

#### 编写Deployment文件

> Deployment：声明了 Pod 的模板和控制 Pod 的运行策略，适用于部署无状态的应用程序。您可以根据业务需求，对 Deployment 中运行的 Pod 的副本数、调度策略、更新策略等进行声明。

```yaml
apiVersion: apps/v1         # 指定的api版本，要符合kubectl apiVersion规定，v1是稳定版，必选参数
kind: Deployment            # k8s资源类型，采用Deployment部署，必选参数
metadata:                   # 资源的元数据语句块，是针对kind对应资源的全局属性，必选参数
  name: flask-script-deployment    # 自定义名称，基于此yaml文件创建的pod，都带有此处名称的前缀
  namespace: default        # 默认命名空间
spec:                       # 规格语句块
  replicas: 2               # 目标副本数，创建的pod维持2个副本，也就是两个pod
  selector:                 # 标签选择器，deployment操作pod，就是通过此功能实现的
    matchLabels:            # 匹配标签语句块，在此语句块中可以写单个标签，也可以写标签集合
      app: flask-script            # 单个标签，key是app，value是flask-script
  template:                 # 创建pod的模版语句块
    metadata:               # 此模版的元数据语句块
      labels:               # 基于此模版创建的pod标签组
        app: flask-script          # pod标签，这里的标签需要跟spec.selector.matchLabels中的标签保持一致
    spec:                   # pod模版的规格语句块
      containers:           # 容器语句块
          - name: flask-script-image      # 容器名称，可以自定义
            image: ccr.ccs.tencentyun.com/flask_script/flask_script:v1  # 容器镜像地址
            ports:          # 暴漏容器端口
            - containerPort: 8080  # 定义8080端口


```

#### 编写Service文件

> Service：k8s 使用 service 来实现服务发现。运行了一个deployment类型的应用, 如何通过浏览器访问到这个应用?
>
> Pod IP仅仅是集群内可见的虚拟IP, 外部无法访问，Pod IP会随着Pod的销毁而消失, 当ReplicaSet对Pod进行动态伸缩时, Pod IP可能随时随地都会发生变化, 这样对于我们访问这个服务带来了难度，隐私通过pod的ip去访问服务, 基本上是不现实的,
>
> 解决方案就是新的资源 (Service) 负载均衡

```yaml
apiVersion: v1                  # 指定的api版本，要符合kubectl apiVersion规定，v1是稳定版，必选参数
kind: Service                   # k8s资源类型，采用Service部署，必选参数
metadata:                       # 资源的元数据语句块，是针对kind对应资源的全局属性，必选参数
  name: flask-script-service    # 自定义名称，Service资源名字
  namespace: default            # 默认命名空间
spec:                           # 规格语句块
  ports:                        # 端口配置语句块
    - port: 5000                # 本Service的端口，可以自定义
      targetPort: 8080          # 目标端口号，也就是根据selector基于标签app: nginx选中的pod里容器的端口
      nodePort: 31112           # 对外暴露端口
  selector:                     # 选择器，基于标签进行选择
    app: flask-script           # 标签对，指向pod，key值为nginx
  type: NodePort                # service模式，有ClusterIP,NodePort,Loabalancer等
```

#### k8s部署应用

> 将Deployment文件和Service文件推送到集群主节点中

![image-20220902102412401](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902102412401.png)

> 创建deployments
>
> kubectl apply -f  flask_dep.yaml

![image-20220902102808382](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902102808382.png)

> 创建Service

![image-20220902103037205](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902103037205.png)

> Postman访问

![image-20220902103138623](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902103138623.png)

### 四、Jenkins pipeline构建自动化DevOps流水线

#### 编写pipeline

```javascript
pipeline {
    agent {
        label '	aliyun-node'
    }
    
    environment {
        imagename = "flask_script"
        tag = "v${BUILD_NUMBER}"
        GIT_REPOSITORY_URL = "https://github.com/zhzhang12138/flask_script.git"
        GIT_CREDENTIALS_ID = "Github"
         
        DOCKER_REPOSITORY_URL = "ccr.ccs.tencentyun.com/flask_script"       
    }
    
    stages {
        stage('Pull code') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: "${GIT_REPOSITORY_URL}"
            }
        }
        
        stage('Clean local history images') {
            steps {
                sh 'docker images | grep ${imagename} | awk \'{print $1":"$2}\' | xargs docker rmi || echo "DONE"'
            }
        }
        
        
         stage('Build docker image') {
            steps {
                timestamps{
                    sh 'docker build --tag=${imagename}:${tag} -q .'
                    sh 'docker tag ${imagename}:${tag} ${DOCKER_REPOSITORY_URL}/${imagename}:${tag}'
                    sh 'docker tag ${imagename}:${tag} ${DOCKER_REPOSITORY_URL}/${imagename}:latest'
                }
            }
        }
        
        stage('Push docker image') {
            steps {
                sh 'docker login ccr.ccs.tencentyun.com --username=100018328451 --password=Zt081998CJJ'
                sh 'docker push ${DOCKER_REPOSITORY_URL}/${imagename}:${tag}'
                sh 'docker push ${DOCKER_REPOSITORY_URL}/${imagename}:latest'
            }
        }

        stage('Kubectl set image') {
           steps {
               withKubeConfig([credentialsId: 'txunyun']) {
                   sh 'kubectl  set image  deployment/flask-script  flask-script-image=ccr.ccs.tencentyun.com/flask_script/flask_script:${tag}'
               }
           }
       }
        
        
    }
}
```

#### 构建

![image-20220902111922786](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902111922786.png)



![image-20220902112159363](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902112159363.png)











